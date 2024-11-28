[toc]

# Raft

针对`Multi Poxos`的提出工程化改进，并提供工程化实现。

### 成员

* `leader`主节点，负责接受与响应用户请求，并向从节点发送日志记录，以及收集从节点反馈。
* `follower`从节点，负责响应`leader`日志复制和`condidate`竞选主节点请求。

* `condidate`候选人节点，竞选`leader`时的临时状态，竞选成功后成为`leader`，失败后成为`follower`。

### 选举

<img src=".\assets\image-20240519210930046.png" alt="image-20240519210930046" style="zoom:50%;" />

* 节点间通过RPC通信，`AppendEntries RPC`用于日志同步、`RequestVote RPC`用于主节点选举，心跳信号为内容为空的`AppendEntries RPC`。
* 每个节点都有单调递增的任期编号`curr_term`，==视为逻辑时钟，用于检测过期信息==，并定期与其它节点交换任期信息，当发现更高任期的节点(==当前节点选举失败、所在分区被合并==)，则将自己任期更新，并成为`follower`节点。

#### 流程

* 主节点定时广播心跳信号，宣誓主权。

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #tickHeartbeat
    // 定时心跳宣示主权
    func (r *raft) tickHeartbeat() {
        // 递增心跳计数器
    	r.heartbeatElapsed++
    	// 满足发送心跳时间间隔
    	if r.heartbeatElapsed >= r.heartbeatTimeout {
    		r.heartbeatElapsed = 0
    		if err := r.Step(pb.Message{From: r.id, Type: pb.MsgBeat}); err != nil {
    		}
    	}
    }
    ```



* 当`follower`在超时后未收到主节点心跳信号，将成为`pre_condidate`，如果节点日志不落后于过半节点(通过投票实现，预选举可以投出多票)，将成为`condidate`，之后为自己投一票，并将`curr_term +=1`，向其它节点发送`RequestVote `请求。==etcd引入预选举机制，如果节点日志不落后于过半节点，才有可能成为新主节点，否则将放弃竞选，减少无效选举次数==。

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #tickElection
    // 超时触发选举/预选举流程
    func (r *raft) tickElection() {
    	r.electionElapsed++
    	// 可以被提升为leader && 候选等待超时
    	if r.promotable() && r.pastElectionTimeout() {
    		r.electionElapsed = 0
    		if err := r.Step(pb.Message{From: r.id, Type: pb.MsgHup}); err != nil {
    		}
    	}
    }
    
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #campaign
    // 开始选举/预选举
    func (r *raft) campaign(t CampaignType) {
    	// etcd引入预选举，如果节点日志不落后于过半节点，才有可能成为新主节点，
        // 否则将放弃竞选，减少无效选举次数
    	if t == campaignPreElection {
    		r.becomePreCandidate()
    		voteMsg = pb.MsgPreVote
    		term = r.Term + 1
    	} else {
            // 切换到Candidate状态，term已在预先选举中递增
    		r.becomeCandidate()
    		voteMsg = pb.MsgVote
    		term = r.Term
    	}
        // 向集群中的所有节点发送信息，请求投票
    	for _, id := range ids {
    		if id == r.id {
    			// 候选人为自己投一票
    			r.send(pb.Message{To: id, Term: term, Type: voteRespMsgType(voteMsg)})
    			continue
    		}
    		// 向集群中的其它节点发送投票请求
    		r.send(pb.Message{To: id, Term: term, Type: voteMsg, Index: r.raftLog.lastIndex(), LogTerm: r.raftLog.lastTerm(), Context: ctx})
    	}
    }
    ```

    

* 其余节点处，如果`pre_condidate/condidate`的日志不落后于当前节点（`condi.LogTerm > MyTerm || (condi.LogTerm == MyTerm && condi.Index >= MyIndex)`），将响应`RequestVote `请求成功信号，否则响应失败信号，==节点在一个任期内，只有一次正式选举投票机会，但有多次预选举投票机会==。

```go
// https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #Step
// 节点响应预选举/选举拉票请求
func (r *raft) Step(m pb.Message) error {
    case pb.MsgVote, pb.MsgPreVote:
			// 已为m.From节点投过票 	
		canVote := r.Vote == m.From ||
			// 当前未投过票并且集群中无主节点
			(r.Vote == None && r.lead == None) ||
			// pre_condidate任期高于自己
			(m.Type == pb.MsgPreVote && m.Term > r.Term)
    
		// 当前节点可以投票 && m.From节点的termId,logindex不落后当前节点
   	    // isUpToDate: m.LogTerm > MyTerm || (m.LogTerm == MyTerm && m.Index >= MyIndex)
		if canVote && r.raftLog.isUpToDate(m.Index, m.LogTerm) {
			//  m.From数据不落后于当前节点，投赞成票
			r.send(pb.Message{To: m.From, Term: m.Term, Type: voteRespMsgType(m.Type)})
            // 预投票不产生真正主节点，不消耗票数，可以投多票
			if m.Type == pb.MsgVote {
				r.Vote = m.From
			}
		} else {
			// m.From数据落后于当前节点，投反对票
			r.send(pb.Message{To: m.From, Term: r.Term, Type: voteRespMsgType(m.Type), Reject: true})
		}
}
```



* 获得过半票的`pre_condidate`将成为`condidate`，开始真正的竞选。获得过半票的`condidate`将成为`condidate`，成功后成为新的`leader`，并广播心跳，宣示主权，其余`condidate`将收到新`leader`的心跳信号，转变为`follower`。

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #stepCandidate
    // 响应投票结果
    func stepCandidate(r *raft, m pb.Message) error {
        gr, rj, res := r.poll(m.From, m.Type, !m.Reject)
        switch res {
            // 赢得投票
        case quorum.VoteWon:
            // 如果赢得的是预选举，将进入真正的选举
            if r.state == StatePreCandidate {
                r.campaign(campaignElection)
            } else {
                // 赢得真正的选举，成为主节点，广播心跳宣示主权
                r.becomeLeader()
                r.bcastAppend()
            }
        case quorum.VoteLost:
            // 输掉预选举、或者真正选举，变回从节点
            r.becomeFollower(r.Term, None)
        }
    }
    ```

    

* 为避免多节点同时成为`pre_condate`，并在预选举成功后成为`condate`，导致分票，选举失败，Raft为==每个节点设置随机的主节点心跳检测超时时间==`[Timeout, 2*Timeout)`。

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go
    func (r *raft) resetRandomizedElectionTimeout() {
    	r.randomizedElectionTimeout = r.electionTimeout + globalRand.Intn(r.electionTimeout)
    }
    ```

    

#### 安全性

* ==利用`log`的连续无空洞特性，某个`entry`被确认，则之前的`entry`都被确认。如果某个`entry`被确认写入大部分从节点，则保证过半从节点与主节点保持一致==；

* `follower`只为最新数据不晚于自己的`condidate`投票，保证只有和主节点拥有相同数据的`condidae`才能成为`leader`，==避免`leader`从`follower`同步数据==；

    ```pseudocode
    if my_termid > condi_termid:
      return refuse
    elif my_termid == condi_termid and my_logindex > condi_logindex:
      return  refuse
    else:
      return accept
    ```

    

### log复制

* 日志log中每个`entry`包含提交该`entry`的任期、在`log`中下标和命令内容`<termId, logindex, cmd>`。

    ```go
    // etcd#raft.pb.go#Entry
    type Entry struct {
    	Term  uint64    
    	Index uint64    
    	Data  []byte    
    }
    ```

* 每个节点包含以下数据：`curr_termid`当前任期、`log[]`日志、`commitInex`log中已提交`entry`最大下标；

    主节点在此基础上，还拥有`nextindex[]`表示每个`follower`里面下一次应该写入的下标(类似TCP中的`ack`)。

* `Message`是消息的抽象，用于各个节点间通讯，包含各种消息所需要的字段：当前任期、待发送的多个日志信息、前，

    ```go
    // etcd#raft.pb.go#Message
    type Message struct {
    	Term uint64      
    	// 第一条Entry的Term值
    	LogTerm    uint64   
    	// 第一条Entry的logindex-1
    	Index      uint64   
    	// 需要存储的日志信息
    	Entries    []Entry  
    }
    ```

    

#### 流程

* (0)-> 新写入操作到来，设置`entry`的`termid, logindex`，并将`entey`写入主节点log，该`entry`状态记为`uncommited`，最后广播日志消息；

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #stepLeader
    // 主节点写入日志，并广播日志
    func stepLeader(r *raft, m pb.Message) error {
    	switch m.Type {
    	case pb.MsgProp:
            //  主节点写入日志
    		if !r.appendEntry(m.Entries...) {
    			return ErrProposalDropped
    		}
            // 向从节点广播日志
    		r.bcastAppend()
    		return nil
        }
    }
    
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #appendEntry
    // 主节点设置消息的termid, logindex后写入自身log
    func (r *raft) appendEntry(es ...pb.Entry) (accepted bool) {
    	li := r.raftLog.lastIndex()
        // 设置termid, 与递增logindex
    	for i := range es {
    		es[i].Term = r.Term
    		es[i].Index = li + 1 + uint64(i)
    	}
        // 写入主节点log
        li = r.raftLog.append(es...)
    	r.send(pb.Message{To: r.id, Type: pb.MsgAppResp, Index: li})
    	return true
    }
    ```

    

* (1)->主节点向从节点发送`AppendEntries`请求，携带当前任期、log中`nextindex[i]~logindex`之间`entry`内容，以及第`nextindex[i]`个`entry`的前一个`entry`的下标和任期：`<termid, enties, prev_logindex, prev_termid>`

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #bcastAppend
    // 向除自身外节点广播日志消息
    func (r *raft) bcastAppend() {
    	r.trk.Visit(func(id uint64, _ *tracker.Progress) {
    		if id == r.id {
    			return
    		}
    		r.sendAppend(id)
    	})
    }
    
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #maybeSendAppend
    // 主节点构造需要发生的数据
    func (r *raft) maybeSendAppend(to uint64, sendIfEmpty bool) bool {
    	pr := r.trk.Progress[to]
    	// pr.Next表示follower的日志中下一次应该写入的下标(类似TCP中的ack)
        // 当前消息应该从followerlog的pr.Next处开始写入
        // pr.Next-1表示follower的日志中，开始写入位置前一个entey下标
    	lastIndex, nextIndex := pr.Next-1, pr.Next
        // lastTerm表示follower开始写入位置前一个entey里面的任期
        // (lastTerm,lastIndex)将用于确认新写入日志能与原有日志拼接出完整日志
    	lastTerm, errt := r.raftLog.term(lastIndex)
    
    	var ents []pb.Entry
    	var erre error
        // 发送消息包含log中nextindex到logindex之间entry内容，
    	if pr.State != tracker.StateReplicate || !pr.Inflights.Full() {
    		ents, erre = r.raftLog.entries(nextIndex, r.maxMsgSize)
    	}
    	if err := pr.UpdateOnEntriesSend(len(ents), uint64(payloadsSize(ents)), nextIndex); err != nil {
    		r.logger.Panicf("%x: %v", r.id, err)
    	}
    }
    ```

    

* (2)->从节点收到消息后，通过检查本地日志中是否存在`lastIndex`对应的节点，判断收到的日志能否写入，并且不会造成空洞。如果`lastIndex`对应节点存在，则找到新收到日志中`entries[lastIndex:]`与本地日志`log[lastIndex:]`不匹配的第一个节点`x`，本地日志`log[lastIndex+1:x-1]`与要添加日志`entries[0:x-lastIndex-2]`之间一致，不用覆盖，只需要将`entries[x-lastIndex-1:]`追加到本地日志末尾，之后更新`commitIndex`，并返回匹配，主节点更新`nextindex[i]`；如果不存在则返回不匹配；

    <img src=".\assets\image-20240522024841477.png" alt="image-20240522024841477" style="zoom:33%;" />

    以上图为例：主节点将`log[6:9]`处日志复制给从节点，此时`lastIndex=5, lastterm=4`。从节点node0处不存在`logindex=5, term=4`的节点，无法将`log[6:9]`复制到本地，同时不产生空洞；从节点node1处存在`logindex=5, term=4`的节点，可以进行复制，然后找到主节点`log[6:9]`与本地`log[6:]`之间第一个差异点的的下标`x=8`，最后将主节点的`log[8:9]`覆盖到从节点`log[8:]`。

    

    ```go
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #stepFollower
    // 从节点收到消息后尝试写入log
    func stepFollower(r *raft, m pb.Message) error {
    	switch m.Type {
    	case pb.MsgApp:
    		r.electionElapsed = 0
    		r.lead = m.From
    		r.handleAppendEntries(m)
    	return nil
    }
        
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #handleAppendEntries
    // 从节点判断接收到entry能否加到自身log中，获得完整准确的log
    func (r *raft) handleAppendEntries(m pb.Message) {
        // 请求的日志索引小于follower已提交的日志索引，不需要重复写入
    	if m.Index < r.raftLog.committed {
    		r.send(pb.Message{To: m.From, Type: pb.MsgAppResp, Index: r.raftLog.committed})
    		return
    	}
        // 尝试将leader的日志追加到自己的日志中
    	if mlastIndex, ok := r.raftLog.maybeAppend(m.Index, m.LogTerm, m.Commit, m.Entries...); ok {
    		r.send(pb.Message{To: m.From, Type: pb.MsgAppResp, Index: mlastIndex})
    		return
    	}
        // 不能追加到消息尾部，需要找到主从节点的公共log节点x，
        // 之后主节点发送log[x:]到从节点，从节点从x处开始覆盖本地log
    	hintIndex := min(m.Index, r.raftLog.lastIndex())
    	hintIndex, hintTerm := r.raftLog.findConflictByTerm(hintIndex, m.LogTerm)
    	r.send(pb.Message{
    		To:         m.From,
    		Type:       pb.MsgAppResp,
    		Index:      m.Index,
    		Reject:     true,
    		RejectHint: hintIndex,
    		LogTerm:    hintTerm,
    	})
    }
        
    // https://github.com/etcd-io/raft/blob/v3.6.0-alpha.0/raft.go #maybeAppend
    // 尝试将日志追加到当前日志末尾
    func (l *raftLog) maybeAppend(index, logTerm, committed uint64, ents ...pb.Entry) (lastnewi uint64, ok bool) {
        // index: ents中第一个entry的下标-1
        // 如果现有日志和要追加的日志无交集，无法添加到现有日志末尾
    	if !l.matchTerm(index, logTerm) {
    		return 0, false
    	}
    	lastnewi = index + uint64(len(ents))
        // 找到现有日志和要添加日志之间第一个冲突点下标ci
        // 本地日志log[index+1:ci-1]与要添加日志ents[0:ci-index-2]之间一致，不用覆盖
        // 将ents[ci-index-1]添加到log末尾即可
    	ci := l.findConflict(ents)
        offset := index + 1
        l.append(ents[ci-offset:]...)
        // 提交新写入日志
        l.commitTo(min(committed, lastnewi))
    	return lastnewi, true
    }
    ```

    

* (3)->如果消息中entries不能直接添加到本地Log末尾，说明entries与本地log无公共节点，将不断尝试`nextindex[i]-=1`，以回溯找寻公共节点，反馈后将发送新一轮的`nextindex[i]~logindex`之间`entry`，重试步骤`(1), (2), (3)`，直至返回匹配；

    ```go
    // 尝试找到本地log与消息中entries的公共节点    
    func (l *raftLog) findConflictByTerm(index uint64, term uint64) (uint64, uint64) {
        // 向前找到一个小于或等于leader日志任期的日志索引
    	for ; index > 0; index-- {
    		if ourTerm, err := l.term(index); err != nil {
    			return index, 0
    		} else if ourTerm <= term {
    			return index, ourTerm
    		}
    	}
    	return 0, 0
    }
    ```

    

* (4)->主节点收到过半从节点写入成功，该`entry`状态记为`commited`，向客户返回成功；

```pseudocode
func (r *raft) maybeCommit() bool {
	// 获得已保存到过半节点的日志的最大索引
	mci := r.trk.Committed()
	// 提交该索引
	return r.raftLog.maybeCommit(mci, r.Term)
}
```

#### 安全性保障

* 保证主从log中具有相同`termId, logindex`的`entry`的`cmd`相同；

    保证主从log中，如果两边存在某个`entry`的`termId, logindex`匹配，则该`entey`之前的`entry`都匹配；

* ==所有节点都会拥有一致的状态机输入序列，各个节点通过一致的初始状态 + 一致的状态机输入序列得到一致的最终状态。==

#### 遗留数据处理

* 当新主节点选举后，主节点可能包含未满足过半从节点确认，未提交的日志。新主节点首先将该部分日期复制到过半从节点，但是并不提交。而是当有新消息到来后，通过提前当前日志的方式，利用log连续无空洞特性，之前复制的遗留日志将一并被提交。

* 为什么不能主动提交遗留日志：假设主节点`A`上有遗留日志`m0`，节点B有其他遗留日志`m1`：`A(m0), B(m1), C()`。如果复制`m0`时未覆盖节点`B`上的`m1`，得到`A(m0), B(m1), C(m0)`，然后提交`m0`。当节点`B`选举成为主节点后，复制`m1`时覆盖了`A`上的`m0`：`A(m1), B(m1), C(m1)`导致提交后的日志`m0`丢失。

    如果不主动提交遗留日志，在节点`A`成为主节点，复制`mo`，`B`成为主节点，复制消息`m1`后，`A(m1), B(m1), C(m1)`。复制来到消息后：`A(m1,m2), B(m1,m2), C(m1,m2)`,然后主动提交`m2`，`m1`被附带确认，此时消息`m0`丢失，但系统保持状态一致。

### Log瘦身

<img src=".\assets\image-20240520014424603.png" alt="image-20240520014424603" style="zoom:50%;" />

* Raft通过建立快照，保存当前时刻状态机(变量)的最终值，类似于Redis追加日志的整理，实现历史数据清理。

### 最终一致性

* raft无法保证实时主从一致性，只能保证最终一致性。通过限制只能从主节点读取，保证一致的最新的数据视图。
* 通过主节点的心跳信号（心跳信号处理简单，但是会瞬时增大硬件资源开销），或者手动发送空白`appendEntries`（日志复制处理复杂，实质上属于背压机制，控制了并发度），如果收到过半从节点响应，证明当前节点是主节点，可以从当前节点读取值，并返回。如果不能保证当前节点是主节点，将读取请求转发到主节点。

# `multi Pasox`与`Raft`

### 相同点

* 从所有节点中选出主节点，接受所有的写操作，并将日志发送给从节点;
* 过半节点复制日志后，该日志即可提交，所有节点将该日志中的命令应用于他们的状态机；
* 如果主节点宕机，通过过半选举的方式选出一个新的主节点；
* 满足状态机安全性：节点间如果`entry`的任期和下标相同，则该`entry`完全一致；
* 满足领导完整性：后续主节点将继承之前主节点提交日志，不做修改。

### 不同点

* `Raft`算法给出了大量的实现细节(如何选举主节点、如何保证主从日志一致、如何为日志瘦身)；

* `Raft`写日志操作必须是串行的，只有前一个请求完成后，才能处理下一个请求；

    `Multi-Paxos`可以并发修改日志，允许日志不按顺序提交，如果当前请求未能抢占到锁，将帮助之前获得锁请求完成日志复制；

* `Raft`利用log完整无空洞的特性，保证过半从节点数据完整，新选举的`leader`数据也是完整的，保证日志只能从`leader`流向`follower`;

    `Multi-Paxo`复制请求顺序是随机的，日志存在空洞，新`leader`产生后，需要重新对每个未提交的日志进行确认，向其它从节点学习缺失，但是可以被提交的日志，优点是写入并发性能提高，提高吞吐量。