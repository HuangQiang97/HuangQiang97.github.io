[toc]

## 8:Gui

```python
import tkinter as tk
from tkinter import filedialog, messagebox

# 参考： https://www.cnblogs.com/shwee/p/9427975.html

#根窗体就是画板，在tkinter中则是Toplevel，画布就是tkinter中的容器（Frame），
# 画板上可以放很多张画布（Convas），tkinter中的容器中也可以放很多个容器，
# 绘画中的构图布局则是tkinter中的布局管理器（几何管理器），绘画的内容就是tkinter中的一个个小组件

def gui():
    # 创建窗口
    window = tk.Tk()
    window.title('window title')
    # w*h
    window.geometry('1000x1000')
    # weight数据
    click_count=None
    entry_input=None
    text=None
    list_items=None
    radio_button=None
    check_button=None

    # 更改label内容
    def click():
        click_count.set(int(click_count.get()) + 1)
        print(click_count.get())
    #构建插件
    click_count = tk.StringVar()
    click_count.set(0)
    # height=2,就是标签有2个字符这么高
    # 动态显示文本，静态文本直接text='msg
    label = tk.Label(window,textvariable=click_count, font=('Arial', 12), width=30, height=2, bg='gray')
    #左上角位置坐标，label.pack()默认放在中间。
    label.place(x=10, y=0)
    # 构建按钮
    b = tk.Button(window, text='click', font=('Arial', 12), width=10, height=1, command=click)
    b.place(x=10, y=50)

    # 编辑text
    def insert():
        input=entry_input.get()
        #在鼠标处插入，'end':在尾部插入
        text.insert('insert',input)
        # x.y ：第x行，第y列，行从1 开始，列从0开始。
        # 1.2-->end
        print(text.get(1.2,'end'))

    # 单行文本输入域
    # show='*' 密码格式，普通文本：show:None
    entry_input = tk.Entry(window, show='*', font=('Arial', 14))
    entry_input.place(x=10, y=100)

    b = tk.Button(window, text='insert', font=('Arial', 12), width=10, height=1, command=insert)
    b.place(x=10, y=150)

    # 多行文本输入
    text = tk.Text(window, height=3,width=20)
    text.place(x=10, y=200)

    # 获得选中项
    def print_list_selection():
        print(list_items.get(list_items.curselection()))

    # 构建下拉列表
    var=tk.StringVar()
    var.set((1,2,3,4))
    list_items = tk.Listbox(window, listvariable=var,height=3)
    # lb.insert(1, 'first')       # 在第一个位置加入'first'字符
    # lb.delete(2)  #删除选项
    list_items.place(x=10, y=250)

    v=tk.Button(window,text='show selected', width=15, height=1, command=print_list_selection)
    v.place(x=10, y=450)

    #输出单选框内容
    def print_button_selection():
        print(radio_button.get())
    # 构建单选框
    radio_button=tk.StringVar()
    r1 = tk.Radiobutton(window, text='A', variable=radio_button, value='A', command=print_button_selection)
    r1.place(x=10, y=500)
    r2 = tk.Radiobutton(window, text='B', variable=radio_button, value='B', command=print_button_selection)
    r2.place(x=10, y=550)

    # 输出确认项
    def print_check_selection():
        print(check_button.get())

    # 构建check button
    check_button = tk.IntVar()
    c = tk.Checkbutton(window, text='check',variable=check_button, onvalue=1, offvalue=0, command=print_check_selection)    # 传值原理类似于radiobutton部件
    c.place(x=10, y=600)

    # 输出滑动条
    # 默认传参v
    def print_scale_selection(v):
        print(v)

    # 构建滑动条
    s = tk.Scale(window, label='scale msg', from_=0, to=10, orient=tk.HORIZONTAL, length=200, showvalue=1,tickinterval=2, resolution=0.1, command=print_scale_selection)
    s.place(x=10, y=650)

    # 构建帧
    frame = tk.Frame(window,bg='green',height=300,width=300)
    frame.place(x=600,y=10)
    # 在帧上构建画布
    canvas = tk.Canvas(frame, bg='red', height=200, width=200)
    # 说明图片位置，并导入图片到画布上
    image_file = tk.PhotoImage(file='pic.gif')  # 图片位置（相对路径，与.py文件同一文件夹下，也可以用绝对路径，需要给定图片具体绝对路径）
    image = canvas.create_image(0, 0, anchor='nw', image=image_file)  # 图片锚定点（n图片顶端的中间点位置）放在画布（250,0）坐标处
    canvas.place(x=10,y=10)

    # 文件操作
    def OpenFile():
        f = filedialog.askopenfilename(title='打开文件', filetypes=[('Python', '*.py *.pyw'), ('All Files', '*')])
        print(f)
        # 可使用os 模块运行文件
    def SaveFile():
        f = filedialog.asksaveasfilename(title='保存文件', initialdir='d:\mywork', initialfile='hello.py')
        print(f)

    def do_job():
        print('do something')

    # 构建按钮栏
    menubar = tk.Menu(window)
    # 菜单项（默认不下拉 tearoff=0，下拉内容包括Open，Save，Exit功能项）
    filemenu = tk.Menu(menubar, tearoff=0)
    # 将上面定义的空菜单命名为File，放在菜单栏中
    menubar.add_cascade(label='File', menu=filemenu)
    # 在File中加入Open、Save等小菜单，即我们平时看到的下拉菜单，每一个小菜单对应命令操作。
    filemenu.add_command(label='Open', command=OpenFile)
    filemenu.add_command(label='Save', command=SaveFile)
    # 添加一条分隔线
    filemenu.add_separator()
    # 用tkinter里面自带的quit()函数
    filemenu.add_command(label='Exit', command=window.quit)

    # 创建一个Edit菜单项（默认不下拉，下拉内容包括Cut，Copy，Paste功能项）
    editmenu = tk.Menu(menubar, tearoff=0)
    # 将上面定义的空菜单命名为 Edit，放在菜单栏中，就是装入那个容器中
    menubar.add_cascade(label='Edit', menu=editmenu)
    # 同样的在 Edit 中加入Cut、Copy、Paste等小命令功能单元，
    editmenu.add_command(label='Cut', command=do_job)
    editmenu.add_command(label='Copy', command=do_job)
    editmenu.add_command(label='Paste', command=do_job)

    #创建第二级菜单，即菜单项里面的菜单
    submenu = tk.Menu(filemenu)
    # 给放入的菜单submenu命名为Import
    filemenu.add_cascade(label='Import', menu=submenu, underline=0)  #

    # 创建第三级菜单命令
    submenu.add_command(label='Submenu', command=do_job)

    # 创建菜单栏完成后，配置让菜单栏menubar显示出来
    window.config(menu=menubar)

    # grid 实现布局
    # row为行，colum为列，padx就是单元格左右间距，pady就是单元格上下间距，
    # ipadx是单元格内部元素与单元格的左右间距，ipady是单元格内部元素与单元格的上下间距。
    canvas = tk.Canvas(window, bg='gray', height=200, width=200)
    canvas.place(x=600,y=400)
    for i in range(3):
        for j in range(3):
            tk.Label(canvas, text=1).grid(row=i, column=j, padx=10, pady=10, ipadx=10, ipady=10)

    # 创建消息框 bool
    result = messagebox.askokcancel('Python Tkinter', 'ok/cancel')
    print(result)
    # str yes/no
    result = messagebox.askquestion('Python Tkinter', "yes/no?")
    print(result)
    # bool
    result = messagebox.askyesno('Python Tkinter', 'true/flase？')
    print(result)
    # 警告，错误
    messagebox.showinfo('Python Tkinter', 'info')
    messagebox.showwarning('Python Tkinter', 'warning')
    messagebox.showerror('Python Tkinter', 'error')

    #创建一个全新的window
    window_up = tk.Toplevel(window)
    window_up.geometry('300x200')
    window_up.title('Sign up window')

    # window.mainloop就会让window不断的刷新，如果没有mainloop,就是一个静态的window
    # 一个gui程序只能有一个mainloop
    window.mainloop()

if __name__=='__main__':
    gui()

```

## 9: Net

### 9.1:TCP_web

```python
import socket
#  创建一个socket
# AF_INET指定使用IPv4协议，如果要用更先进的IPv6，就指定为AF_INET6。SOCK_STREAM指定使用面向流的TCP协议，
with socket.socket(socket.AF_INET,socket.SOCK_STREAM) as s:
    # 注意参数是一个tuple，包含地址和端口号。
    s.connect(('www.sina.com',80))
    # 发送数据:
    s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')
    buffer=[]
    while True:
        #一次最多接收指定的字节数
        temp=s.recv(1024)
        if temp:
            buffer.append(temp)
        # recv()返回空数据，表示接收完毕
        else:
            break
buffer=b''.join(buffer)
header, html = buffer.split(b'\r\n\r\n', 1)
print(header.decode('utf-8'))
with open('sina.html','w') as f:
    f.write(html.decode('utf-8'))

```

### 9.2:TCP_client

````python
import socket
import os
import struct
import json
# AF_INET指定使用IPv4协议，如果要用更先进的IPv6，就指定为AF_INET6。
# SOCK_STREAM指定使用面向流的TCP协议，收发都要使用二进制数据，注意转换。
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    # 建立连接:参数是一个tuple，包含地址和端口号。
    s.connect(('127.0.0.1', 9999))
    # 接收消息,1024 :一次最多接收指定的字节数
    print(s.recv(1024).decode('utf-8'))

    file_path='sina.html'
    # 得到文件的大小,字节
    filesize_bytes = os.path.getsize(file_path)
    filename = 'recv_'+file_path
    dirc = {
        'filename': filename,
        'filesize_bytes': filesize_bytes,
    }
    # 将字典转换成字符串
    head_info = json.dumps(dirc)
    # 将字符串的长度打包为2进制
    head_info_len = struct.pack('i', len(head_info))
    # 发送头文件
    s.send(head_info_len)
    s.send(head_info.encode('utf-8'))
    #发送文件
    with open(file_path,'rb') as f:
        s.sendall(f.read())
    # 反馈
    msg=s.recv(1024).decode('utf-8')
    print(msg)

````

### 9.3:TCP_server

```python
import socket
import threading
import struct
import json
import sys

# 进度条
def process_bar(precent, width=50):
    use_num = int(precent*width)
    space_num = int(width-use_num)
    precent = precent*100
    print('[%s%s]%d%%'%(use_num*'#', space_num*' ',precent),file=sys.stdout,flush=True, end='\r')

def tcplink(client,addr):
    client.send(b'welcome',)
    # 接受头文件数据
    head_length=struct.unpack('i', client.recv(4))[0]
    head=client.recv(head_length)
    head_dir = json.loads(head.decode('utf-8'))
    filesize = head_dir['filesize_bytes']
    filename = head_dir['filename']
    # 接受文件
    recv_size=0
    with open(filename,'w') as f:
        while True:
            temp=filesize-recv_size
            #recv() 方法在未收到消息下会阻塞当前线程
            process_bar(recv_size/filesize)
            if temp>1024:
                data=client.recv(1024)
                recv_size+=1024
                # 收发都是二进制数据流，要注意转换
                f.write(data.decode('utf-8'))
            else:
                data=client.recv(temp)
                # 收发都是二进制数据流，要注意转换
                f.write(data.decode('utf-8'))
                break
    client.send('trans finished'.encode('utf-8'))
    # 关闭连接
    client.close()
    print(addr,' closed')

# 一个Socket依赖4项：服务器地址、服务器端口、客户端地址、客户端端口来唯一确定一个Socket。
with socket.socket(socket.AF_INET,socket.SOCK_STREAM) as s:
    # 监听端口
    # # 要绑定监听的地址和端口。服务器可能有多块网卡，可以绑定到某一块网卡的IP地址上，也可以用0.0.0.0绑定到所有的网络地址，还可以用127.0.0.1绑定到本机地址。
    s.bind(('127.0.0.1', 9999))
    # 调用listen()方法开始监听端口，
    # 传入的参数指定等待连接的最大数量,实现链接多个对象
    s.listen(5)
    print('Waiting for connection...')
    while True:
        # 接受链接,accept()会等待并返回一个客户端的连接
        client,addr=s.accept()
        # 创建新线程来处理TCP连接:
        t=threading.Thread(target=tcplink,args=(client,addr))
        t.start()
```

### 9.4:UDP_client

```python
import socket
#SOCK_DGRAM指定了这个Socket的类型是UDP。
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    for data in [b'Michael', b'Tracy', b'Sarah']:
        # 不需要调用connect()，直接通过sendto()给服务器发数据：
        # 发送数据,无连接，发送时指明收信方
        s.sendto(data, ('127.0.0.1', 9999))
        # 接收数据:
        print(s.recv(1024).decode('utf-8'))

```

### 9.5:UDP_server

```python
import socket
import threading 

# 使用UDP协议时，不需要建立连接，只需要知道对方的IP地址和端口号，就可以直接发数据包。
#SOCK_DGRAM指定了这个Socket的类型是UDP。
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    # 绑定端口,服务器绑定UDP端口和TCP端口互不冲突.
    s.bind(('127.0.0.1', 9999))
    while True:
        # 需要调用listen()方法，而是直接接收来自任何客户端的数据：
        # recvfrom()方法返回数据和客户端的地址与端口
        data,addr=s.recvfrom(1024)
        print('Received from %s:%s.' % addr)
        s.sendto(b'Hello, %s!' % data, addr)


```

## 10:Mail

参考：[Send_Recv_Mail][Mail_Link]

### 10.1:Send

```python
# 流程： 发件人 -> MUA ---(SMTP)---> MTA---(SMTP)--->MTA ->  MDA <--(POP3/IMAP)--- MUA <- 收件人
# 要编写程序来发送和接收邮件，本质上就是：编写MUA把邮件发到MTA；编写MUA从MDA上收邮件。

# SMTP是发送邮件的协议，Python内置对SMTP的支持，可以发送纯文本邮件、HTML邮件以及带附件的邮件。
# Python对SMTP支持有smtplib和email两个模块，email负责构造邮件，smtplib负责发送邮件。

from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.utils import parseaddr, formataddr

import smtplib
    
# 编码
def _format_addr(s):
    name, addr = parseaddr(s)
     # 如果包含中文，需要通过Header对象进行编码
    return formataddr((Header(name, 'utf-8').encode(), addr))

# 发件人信息
from_addr = '6781@qq.com'
password = 'password'
# 收件人信息
to_addr = '6781@qq.com'
# SMTP 服务器
smtp_server = 'smtp.qq.com'
# 163：465或者994 QQ: 465或587
SSL_part=587

# 一个邮件对象就是一个Messag对象，如果构造一个MIMEText对象，就表示一个文本邮件对象，
# 如果构造一个MIMEImage对象，就表示一个作为附件的图片，要把多个对象组合起来，就用MIMEMultipart对象，
# 而MIMEBase可以表示任何对象。
# Message
# +- MIMEBase
#    +- MIMEMultipart
#    +- MIMENonMultipart
#       +- MIMEMessage
#       +- MIMEText
#       +- MIMEImage

# ---------------------------纯文本-------------------------------------
# 第一个参数就是邮件正文，第二个参数是MIME的subtype，plain表示纯文本,html表示HTML页面，最终的MIME就是'text/plain'，用utf-8编码保证多语言兼容性。
# 发送HTML邮件，在构造MIMEText对象时，把HTML字符串传进去，再把第二个参数由plain变为html 
# msg = MIMEText('hello, send by Python...', 'plain', 'utf-8')
# msg = MIMEText('<html><body><h1>Hello</h1>'+'<p>send by <a href="http://www.python.org">Python</a>...</p>'+'</body></html>', 'html', 'utf-8')

# ---------------------------含附件-------------------------------------
# 创建MIMEMultipart()实例，然后构造附件,如果有多个附件，可依次构造.
# MIMEMultipart(alternative) :表示在收件方如果html无法正常解析，就是用纯文本。发件方要同时加入plain html
# MIMEMultipart()            :只发送plain或者html
msg = MIMEMultipart('alternative')
# 邮件正文是MIMEText
msg.attach(MIMEText('send with file...', 'plain', 'utf-8'))
# HTML内嵌图片要把图片当附件传入，利用编号调用，编号在附件的文件头中确定
msg.attach(MIMEText('<html><body><h1>Hello</h1>' +'<p><img src="cid:1"></p>' +'</body></html>', 'html', 'utf-8'))



# 添加附件就是加上一个MIMEBase，从本地读取文件:
with open('Mail\\pdf.pdf', 'rb') as f:
    # 设置附件的MIME和文件名。
    mime = MIMEBase('doc', 'pdf', filename='show_file_name.pdf')
    # 加上必要的头信息:
    mime.add_header('Content-Disposition', 'attachment', filename='show_file_name.pdf')
    # 文件ID，便于HTML调用
    mime.add_header('Content-ID', '<0>')
    mime.add_header('X-Attachment-Id', '0')
    # 把附件的内容读进来:
    mime.set_payload(f.read())
    # 用Base64编码:
    encoders.encode_base64(mime)
    # 添加到MIMEMultipart:
    msg.attach(mime)

# 读取本地文本文件
with open('Mail\\send.py', 'rb') as f:
    # 构造附件
    att = MIMEText(f.read(), 'base64', 'utf-8')
    # 文件头
    att["Content-Type"] = 'application/octet-stream'
    att["Content-Disposition"] = 'attachment; filename="show_file_name.py"'
    msg.attach(att)

# 添加图片
with open('Mail\\img.png', 'rb') as fp:
    msgImage = MIMEImage(fp.read())
    # 定义图片 ID，在 HTML 文本中引用
    msgImage.add_header('Content-ID', '<1>')
    mime.add_header('X-Attachment-Id', '1')
    # 文件头
    msgImage["Content-Disposition"] = 'attachment; filename="show_file_name.png"'
    msg.attach(msgImage)

# 邮件主题、如何显示发件人、收件人等信息并不是通过SMTP协议发给MTA，而是包含在发给MTA的文本中的
msg['From'] = _format_addr('2812 <%s>' % from_addr)
# 接收的是字符串而不是list，如果有多个邮件地址，用,分隔即可。SSL_pa
msg['To'] = _format_addr('2750 <%s>' % to_addr)

msg['Subject'] = Header('SMTP send', 'utf-8').encode()

# SSL加密传输
# Version 1
# server=smtplib.SMTP(smtp_server,SSL_part)
# server.starttls()
# Version 2
server =smtplib.SMTP_SSL(smtp_server)

# 打印出和SMTP服务器交互的所有信息
server.set_debuglevel(1)
# 登录SMTP服务器
server.login(from_addr, password)
# 可以一次发给多个人，所以传入一个list，邮件正文是一个str，as_string()把MIMEText对象变成str。
# ['****@mail.com','****@mail.com']
server.sendmail(from_addr, [to_addr], msg.as_string())
server.quit()
```

### 10.2:Recv

```python
# POP3协议收取的不是一个已经可以阅读的邮件本身，而是邮件的原始文本，这和SMTP协议很像，SMTP发送的也是经过编码后的一大段文本。
# 要把POP3收取的文本变成可以阅读的邮件，还需要用email模块提供的各种类来解析原始文本，变成可阅读的邮件对象。
# 第一步：用poplib把邮件的原始文本下载到本地；
# 第二部：用email解析原始文本，还原为邮件对象。

from email.parser import Parser
from email.header import decode_header
from email.utils import parseaddr

import poplib

# 输入邮件地址, 口令和POP3服务器地址:
email = '6781@qq.com'
password = 'password'
pop3_server ='pop.qq.com'

# 猜测文字编码
# 首先在message中寻找编码，如果没有，就在header的Content-Type中寻找
def guess_charset(msg):
    charset = msg.get_charset()
    if charset is None:
        content_type = msg.get('Content-Type', '').lower()
        pos = content_type.find('charset=')
        if pos >= 0:
            charset = content_type[pos + 8:].strip()
    return charset

# 解析文本
def decode_str(s):
    # decode_header()返回一个list,这里只返回了第一个元素
    value, charset = decode_header(s)[0]
    if charset:
        value = value.decode(charset)
    return value
# Message对象本身可能是一个MIMEMultipart对象，即包含嵌套的其他MIMEBase对象，
# 递归地打印出Message对象的层次结构
def print_info(msg, indent=0):
    # indent用于缩进显示。
    if indent == 0:
        # 第一层缩进，包含邮件基本信息
        for header in ['From', 'To', 'Subject']:
            value = msg.get(header, '')
            if value:
                if header=='Subject':
                    # 解析主题
                    value = decode_str(value)
                else:
                    # 解析收发件人信息，然后转换成unicode对象。
                    hdr, addr = parseaddr(value)
                    name = decode_str(hdr)
                    # u表示将后面跟的字符串以unicode格式存储
                    value = u'%s <%s>' % (name, addr)
            print('%s%s: %s' % ('  ' * indent, header, value))
    # 解析复合体的载荷
    if (msg.is_multipart()):
        parts = msg.get_payload()
        for n, part in enumerate(parts):
            print('%spart %s' % ('  ' * indent, n))
            print('%s--------------------' % ('  ' * indent))
            # 递归解析只载荷
            print_info(part, indent + 1)
    # 解析单体载荷
    else:
        content_type = msg.get_content_type()
        # html /plain
        if content_type=='text/plain' or content_type=='text/html':
            # 得到文本
            content = msg.get_payload(decode=True)
            # 猜测编码
            charset = guess_charset(msg)
            # 解析
            if charset:
                content = content.decode(charset)
            print('%sText: %s' % ('  ' * indent, content + '...'))

        # 附件
        else:
            print('%sAttachment: %s' % ('  ' * indent, content_type))
            # 下载附件
            file_name = msg.get_filename()
            file_name='Mail\\Recv_'+file_name
            with open(file_name,'wb') as f:
                data=msg.get_payload(decode=True)
                f.write(data)


# 连接到POP3服务器:
server = poplib.POP3_SSL(pop3_server)
# 可以打开或关闭调试信息:
server.set_debuglevel(1)
# 可选:打印POP3服务器的欢迎文字:
print(server.getwelcome().decode('utf-8'))
# 身份认证:
server.user(email)
server.pass_(password)
# stat()返回邮件数量和占用空间:
print('Messages: %s. Size: %s' % server.stat())
# list()返回所有邮件的编号:
# mails包含返回的所有邮件
# 返回一个3元祖(返回信息, 消息列表, 消息的大小)，如果指定index，就只返回指定消息的数据
resp, mails, octets = server.list()
# 可以查看返回的列表类似[b'1 82923', b'2 2184', ...]
print(mails)
# 获取最新一封邮件, 注意索引号从1开始:
index = len(mails)
# lines存储了邮件的原始文本的每一行,
# 可以获得整个邮件的原始文本:
# 获取详细index，设置为已读，返回3元组(返回信息, 消息msgnum的所以内容, 消息的字节数)，
# 如果指定index，就只返回指定消息的数据
resp, lines, octets = server.retr(index)
msg_content = b'\r\n'.join(lines).decode('utf-8')
# 稍后解析出邮件:
msg = Parser().parsestr(msg_content)
print_info(msg)
# 可以根据邮件索引号直接从服务器删除邮件:
# server.dele(index)
# 关闭连接:
server.quit()
```

## 11:SQLite

参考：[SQLite][SQL_Link0]	[SQLite][SQL_Link1]

```python


# 基于表（Table）的一对多的关系就是关系数据库的基础。
# 表是数据库中存放关系数据的集合，一个数据库里面通常都包含多个表，表和表之间通过外键关联。
# 首先连接到数据库，一个数据库连接称为Connection；
# 连接到数据库后，需要打开游标，称之为Cursor，通过Cursor执行SQL语句，然后，获得执行结果。

import sqlite3 as SQL
# 如果文件不存在，会自动在当前目录创建，timeout默认为5
# Connection和Cursor对象，打开后一定记得关闭
conn=SQL.connect('SQL\\test.db',timeout=5)
# 创建在内存上面,执行完任何操作后，都不需要提交事务的(commit)
# conn = sqlite3.connect('"memory:')

# 创建一个Cursor.
cursor=conn.cursor() 
# 执行语句
# 使用Cursor对象执行insert，update，delete语句时，执行结果由rowcount返回影响的行数，就可以拿到执行结果。
# 使用Cursor对象执行select语句时，通过featchall()可以拿到结果集。结果集是一个list，每个元素都是一个tuple，对应一行记录。
# 如果SQL语句带有参数，那么需要把参数按照位置传递给execute()方法，有几个?占位符就必须对应几个参数，参数用元组方式传入。
# REAL:浮点数，存储为IEEE 8byte浮点数
# TEXT:文本字符串，缺省的编码为utf-8
# INT:带符号的整数，根据值的大小，自动存储为1,2,3,4,5,8字节6种
# char :字符

cursor.execute('create table user (id INT  primary key , name varchar(20) not null)')
# 添加列
#cursor.execute('ALTER TABLE user ADD COLUMN age int')

# 插入数据
cursor.execute(r"insert into user (id, name) values (1, 'huang1')")
cursor.execute(r"insert into user (id, name) values (2, 'huang2')")
# 通过rowcount获得插入的行数:
print(cursor.rowcount)
# 获取查询结果集中所有（剩余）的行，返回一个列表。当没有可用的行时，则返回一个空的列表。
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
cursor.execute(r"UPDATE user set name = 'qiang' where ID=2")
# 通过rowcount获得插入的行数:
print(cursor.rowcount)
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
cursor.execute('DELETE from user where ID=2')
# 通过rowcount获得插入的行数:
print(cursor.rowcount)
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
# 关闭游标
cursor.close()
# 该方法回滚自上一次调用 commit() 以来对数据库所做的更改。
# conn.rollback()
# commit-->close
# 提交事务:
conn.commit()
conn.close()
```

## 12:Web

### 12.1:HTTP

>HTTP请求的流程：
>
>步骤1：浏览器首先向服务器发送HTTP请求，请求包括：
>
>方法：GET还是POST，GET仅请求资源，POST会附带用户数据；
>
>路径：/full/url/path；
>
>域名：由Host头指定：Host: [www.sina.com.cn](www.sina.com.cn)
>
>以及其他相关的Header；
>
>如果是POST，那么请求还包括一个Body，包含用户数据。
>
>步骤2：服务器向浏览器返回HTTP响应，响应包括：
>
>响应代码：200表示成功，3xx表示重定向，4xx表示客户端发送的请求有错误，5xx表示服务器端处理时发生了错误；
>
>响应类型：由Content-Type指定，例如：Content-Type: text/html;charset=utf-8表示响应类型是HTML文本，并且编码是UTF-8，Content-Type: image/jpeg表示响应类型是JPEG格式的图片；
>
>以及其他相关的Header；
>
>通常服务器的HTTP响应会携带内容，也就是有一个Body，包含响应的内容，网页的HTML源码就在Body中。
>
>步骤3：如果浏览器还需要继续向服务器请求其他资源，比如图片，就再次发出HTTP请求，重复步骤1、2。

### 12.2:WSGI

>##### hello.py
>
>```python
># environ：一个包含所有HTTP请求头信息的dict对象；
># start_response：一个发送HTTP响应头的函数。
>def application(environ, start_response):
># Header只能发送一次
># 函数接收两个参数，一个是HTTP响应码，一个是一组list表示的HTTP Header，每个Header用一个包含两个str的tuple表示。
>start_response('200 OK', [('Content-Type', 'text/html')])
>body = '<h1>Hello, %s!</h1>' % (environ['PATH_INFO'][1:] or 'web')
>return [body.encode('utf-8')]
>return [b'<h1>Hello, web!</h1>']
>
>```
>
>##### Server
>
>``` ### python
>#负责启动WSGI服务器，加载application()函数：
>from wsgiref.simple_server import make_server
># 导入我们自己编写的application函数:
>from Web.hello import application
>
># 创建一个服务器，IP地址为空，端口是8848，处理函数是application:
>httpd = make_server('', 8000, application)
>print('Serving HTTP on port 8000...')
># 开始监听HTTP请求:
>httpd.serve_forever()
>```

### 12.3:Flask_Web

``` python
### from flask import Flask
from flask import request

app = Flask(__name__)

# Flask通过Python的装饰器在内部自动地把URL和函数给关联起来
@app.route('/', methods=['GET', 'POST'])
def home():
    return '<h1>Home</h1>'

@app.route('/signin', methods=['GET'])
def signin_form():
    return '''<form action="/signin" method="post">
              <p><input name="username"></p>
              <p><input name="password" type="password"></p>
              <p><button type="submit">Sign In</button></p>
              </form>'''

@app.route('/signin', methods=['POST'])
def signin():
    # 需要从request对象读取表单内容：
    if request.form['username']=='admin' and request.form['password']=='password':
        return '<h3>Hello, admin!</h3>'
    return '<h3>Bad username or password.</h3>'

if __name__ == '__main__':
    app.run()
```

### 12.4:MVC

>##### App
>
>```python
># MVC：Model-View-Controller，中文名“模型-视图-控制器”。
># Python处理URL的函数就是C：Controller，Controller负责业务逻辑
># 包含变量的模板就是V：View，View负责显示逻辑，通过简单地替换一些变量，View最终输出的就是用户看到的HTML。
># 包含要用来替换的数据就是M,Model是用来传给View的，这样View在替换变量的时候，就可以从Model中取出相应的数据。
># 把Python代码和HTML代码最大限度地分离了。
># 一定要把模板放到正确的templates目录下，templates和flask_web.py在同级目录下：
>from flask import Flask, request, render_template
>
>app = Flask(__name__)
>
>@app.route('/', methods=['GET', 'POST'])
>def home():
>return render_template('home.html')
>
>@app.route('/signin', methods=['GET'])
>def signin_form():
>return render_template('form.html')
>
>@app.route('/signin', methods=['POST'])
>def signin():
>username = request.form['username']
>password = request.form['password']
>if username=='admin' and password=='password':
># 通过render_template()函数来实现模板的渲染。
>   return render_template('signin-ok.html', username=username)
>return render_template('form.html', message='Bad username or password', username=username)
>
>if __name__ == '__main__':
>app.run()
>```
>
>##### templates
>
>>###### form.html
>
>>```html
>><html>
>><head>
>><title>Please Sign In</title>
>></head>
>><body>
>>{% if message %}
>><p style="color:red">{{ message }}</p>
>>{% endif %}
>><form action="/signin" method="post">
>><legend>Please sign in:</legend>
>><p><input name="username" placeholder="Username" value="{{ username }}"></p>
>><p><input name="password" placeholder="Password" type="password"></p>
>><p><button type="submit">Sign In</button></p>
>></form>
>></body>
>></html>
>>```
>
>>##### home.html
>
>>```html
>><html>
>><head>
>><title>Home</title>
>></head>
>><body>
>><h1 style="font-style:italic">Home</h1>
>></body>
>></html>
>>```
>
>>##### signin-ok.html
>
>>```html
>><html>
>><head>
>><title>Welcome, {{ username }}</title>
>></head>
>><body>
>><p>Welcome, {{ username }}!</p>
>></body>
>></html>
>>```

## 13:Asynchronous IO

参考：[同步/异步 阻塞/非阻塞][asy_link0]	

​				[iterator / generator][asy_link1]	

​				[asyio/cortine][asy_link2]	

```python
# -------------------------------->同步/异步 阻塞/非阻塞------------------------------------

#
# 在一个线程中，CPU执行代码的速度极快，然而，一旦遇到IO操作，如读写文件、发送网络数据时，就需要等待IO操作完成，才能继续进行下一步操作。这种情况称为同步IO。
#
# 使用多线程或者多进程来并发执行代码，为多个用户服务。每个用户都会分配一个线程，如果遇到IO导致线程被挂起，其他用户的线程不受影响，但是系统不能无上限地增加线程。由于系统切换线程的开销也很大，一旦线程数量过多，CPU的时间就花在线程切换上了，结果导致性能严重下降。
#
# loop = get_event_loop()
# while True:
#     event = loop.get_event()
#     process_event(event)
# 当代码需要执行一个耗时的IO操作时，它只发出IO指令，并不等待IO结果，然后就去执行其他代码了。一段时间后，当IO返回结果时，再通知CPU进行处理。
#
# 当遇到IO操作时，代码只负责发出IO请求，不等待IO结果，然后直接结束本轮消息处理，进入下一轮消息处理过程。当IO操作完成后，将收到一条“IO完成”的消息，处理该消息时就可以直接获取IO操作结果。
#
# 在“发出IO请求”到收到“IO完成”的这段时间里，同步IO模型下，主线程只能挂起，但异步IO模型下，主线程并没有休息，而是在消息循环中继续处理其他消息。这样，在异步IO模型下，一个线程就可以同时处理多个IO请求，并且没有切换线程的操作。
#
# 烧水：
#  1 老张把水壶放到火上，立等水开。（同步阻塞）
#  2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）
#  3 老张把水开了会响的水壶放到火上，立等水开。（异步阻塞）
#  4 老张把水开了会响的水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）
#
#  所谓同步异步，只是对于水壶而言：普通水壶，同步，能没有结束前，一直等结果；响水壶，异步，不需要知道该功能结果，该功能有结果后通知（回调通知）。 响水壶可以在自己完工之后，提示老张水开了。 普通水壶同步只能让调用者去轮询自己。
#  所谓阻塞非阻塞，仅仅对于老张而言。 立等的老张，阻塞，（函数）没有接收完数据或者没有得到结果之前，不会返回；看电视的老张，非阻塞，（函数）立即返回，通过select通知调用者。

# -------------------------------->COROUTINE-------------------------------------------

# 子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。
# 所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。
# 子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。
# 协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。有点类似CPU的中断.

# 执行有点像多线程，但协程的特点在于是一个线程执行
# 协程极高的执行效率。子程序切换不是线程切换，而是由程序自身控制，没有线程切换的开销。
# 不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态。
# 协程是一个线程执行，利用多核CPU:多进程+协程，既充分利用多核，又充分发挥协程的高效率。

# yield 与yield
# from:
# yield list:返回整个列表，yield from list:一次返回一个元素。
# yield from:会自动处理大量错误。
# yield from后面必须是子生成器函数
#
# # 子生成器
# def ger_0():
#   loop:
#     x = yield
#     do
#     somthing
#   return result
#
# # 委托生成器
# def ger_1():
#     result = yield from ger_0()
#
# # 调用方
# g = ger_1()
# g.send(0) -->ger_0: x = 0
# g.send(1) -->ger_0: x = 1
# g.send(2) -->ger_0: x = 2
# ger_1: result = ger_0:result

# consumer函数是一个generator
def consumer():
    print('2---------')
    r = ''
    while True:
        print('3---------')
        # 通过yield拿到消息n处理，又通过yield把结果r传回
        # 赋值语句先计算= 右边，由于右边是 yield 语句，
        # 所以yield语句执行完以后，进入暂停，而赋值语句在下一次启动生成器的时候首先被执行；
        # P:sned(None)->C:yield r=''->P:send(1)->C:n=1->C:yield r='200 OK'->P:send(2)
        n = yield r
        if not n:
            print('6---------')
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    print('1---------')
    # 启动生成器
    # 在一个生成器函数未启动之前，是不能传递值进去。
    # 也就是说在使用c.send(n)之前，必须先使用c.send(None)或者next(c)来返回生成器的第一个值
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('4---------')
        print('[PRODUCER] Producing %s...' % n)
        # 切换到consumer执行
        r = c.send(n)
        print('5---------')
        # 拿到consumer处理的结果，继续生产下一条消息
        print('[PRODUCER] Consumer return: %s' % r)
    # 关闭consumer，整个过程结束。
    c.close()

#   1->2->3 ->  4->3->5  ->  4->3->5  ->   4->3->5
c = consumer()
produce(c)


# -------------------------------->ASYNCIO-------------------------------------------

# https://blog.csdn.net/SL_World/article/details/86597738
# asyncio的编程模型就是一个消息循环。我们从asyncio模块中直接获取一个EventLoop的引用，然后把需要执行的协程扔到EventLoop中执行，就实现了异步IO。
# 异步操作需要在coroutine中通过yield from完成；
# 多个coroutine可以封装成一组Task然后并发执行。
import threading
import asyncio
# 把一个generator标记为coroutine类型
@asyncio.coroutine
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    # yield from语法可以让我们方便地调用另一个generator
    # asyncio.sleep()也是一个coroutine，所以线程不会等待asyncio.sleep()，而是直接中断并执行下一个消息循环。
    # 当asyncio.sleep()返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。
    yield from asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
# 两个coroutine是由同一个线程并发执行的。
# 直到循环事件的所有事件都处理完才能完整结束。
loop.run_until_complete(asyncio.wait(tasks))
loop.close()


# 引入了新的语法async和await，可以让coroutine的代码更简洁易读
# @asyncio.coroutine替换为async；
# 把yield from替换为await。

import threading
import asyncio

async def hello():
    print('Hello world! (%s)' % threading.currentThread())
    await asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()


# -------------------------------->调用方-子生成器-委托生成器<-------------------------------


import time
import asyncio

async def taskIO_1():
    print('开始运行IO任务1...')
    await asyncio.sleep(3)
    print('IO任务1已完成，耗时3s')
    return taskIO_1.__name__

async def taskIO_2():
    print('开始运行IO任务2...')
    await asyncio.sleep(2)
    print('IO任务2已完成，耗时2s')
    return taskIO_2.__name__

# 调用方
async def main():
    # 把所有任务添加到task中
    tasks = [taskIO_1(), taskIO_2()]

    # 返回已经完成的任务，完成一个返回一个
    for completed_task in asyncio.as_completed(tasks):
        # 子生成器
        resualt = await completed_task
        print('协程无序返回值：'+resualt)

    # # done：已经完成的任务，pending：未完成任务
    # # 等待任务全部完成才返回
    # done, pending = await asyncio.wait(tasks)
    # for r in done:
    #     print('协程无序返回值：' + r.result())

if __name__ == '__main__':
    start = time.time()
    # 创建一个事件循环对象loop
    loop = asyncio.get_event_loop()
    try:
        # 完成事件循环，直到最后一个任务结束
        loop.run_until_complete(main())
    finally:
        # 结束事件循环
        loop.close()
    print('所有IO任务总耗时%.5f秒' % float(time.time()-start))

# -------------------------------->AIOHTTP-------------------------------------------

# 把asyncio用在服务器端，例如Web服务器，由于HTTP连接就是IO操作，因此可以用单线程+coroutine实现多用户的高并发支持。
# asyncio实现了TCP、UDP、SSL等协议

from aiohttp import web

routes = web.RouteTableDef()

@routes.get('/')
async def index(request):
    await asyncio.sleep(2)
    return web.json_response({
        'name': 'index'
    })

@routes.get('/about')
async def about(request):
    await asyncio.sleep(0.5)
    return web.Response(text="<h1>about us</h1>")

def init():
    app = web.Application()
    app.add_routes(routes)
    web.run_app(app)

init()
```



## 14:其它

### Package

>新建包：在该包文件夹下新建 \_\_init\_\_.py
>
>同文件夹下相互引用：from file_0 import func_0
>
>在pycharm下： make_directory as-->sources Root

引用：

[Mail_link]: https://www.jianshu.com/p/abb2d6e91c1f
[SQL_Link0]:https://www.runoob.com/sqlite/sqlite-python.html
[SQL_Link1]:https://www.liaoxuefeng.com/wiki/1016959663602400/1017801751919456
[asy_link0]:https://www.cnblogs.com/Anker/p/5965654.html
[asy_link1]:https://blog.csdn.net/SL_World/article/details/86507872
[asy_link2]:https://github.com/SparksFly8/Learning_Python/blob/master/coroutine/README.md