# Lua学习

# 1、数据类型

八种类型

**1、nil**  表示无效值 

```Lua
if(type(X) == "nil") then
    如果X是空
```

**2、boolean**     包含true和false

**3、number**    实数，正数等

**4、string** 

使用双引号、单引号或者[["字符串"]]表示

```Lua
--字符串的连接
print("123".."abc")
--字符串的长度#
print(#("123".."abc"))
```

**5、table**

通过{}初始化

映射关系是可以是number或者string

```c++
初始化方式
table = {"apple", "banna", "orange"}  --默认的key从1开始
table = {[1] = "apple", [2] = "banana", ["mas"] = "orange"} //指定key
table["key"] = "value"   
  
```

**6、function**

```Lua
function func_name1(x)
	--do something    
end
//也可以指定变量参数为函数
function func_name2(k, x, func_name1)
    
end
```

**7、协程**

拥有自己独立的栈，局部变量和指令指针等，可以跟其他协程一样共享全局变量

和线程区别：线程可以同时运行多个，一个线程有多个协程，协程任意时刻是能运行一个，且处于运行状态的协程只有被挂起才会停止

**8、userdata**

用户自定义数据，由应用程序或者c/c++语言库所创建，将c/c++中的任意数据类型(通常是struct和指针)存储到lua变量中使用



# 2、常用操作

## 1、变量

默认都是全局变量，使用local变成局部变量

## 2、赋值方式

a,b = 1, 2 // 分别赋值

交换变量，就很神奇

x, y = y, x

函数返回值

a,b = fun()

## 3、循环

```Lua
--while循环
while(true)
do
    --xxx
end

--for循环
for i = 1, 5 do
    --xxx
end
--或者用于遍历table
for k, v in ipairs do
    --xx
end

--do-while..
repeat
    -xx
until()

--if
if() then
    --xx
else if()
       
else
        
end
```

## 4、函数

可以返回任意参数个数

可变参数 ...  使用#arg获取参数个数

```Lua
function test(...)
    local arg = {...} //将可变参数使用table接受
    --#arg表示参数个数
end
```



## 5、运算符

==等于

~= 不等于

^乘幂

and or not   与或非

..连接字符串 #返回字符串长度

## 6、string字符串

字符串转义字符

\101 小写的a， 八进制

\x41 大写的A， 16进制

函数

string.upper(str)    转成大写

string.lower(str) 	转成小写

string.gsub(str, x1, x2, num)  str中x1替换成x2 num次

string.find(str, x, k) 从str的索引k处开始后找x，返回起始和结束索引

string.reserve(arg)  反转

string.format("the vvalue is : %d", 4)  类似printf

string.char(arg) 整形转字符 

string.byte(arg) 字符转换整形

str.len(arg)

str.rep(arg, 2) 重复字符若干次

## 7、迭代器

ipairs ，从下标1开始，直到空结束

pairs，遍历所有元素

for k, v in func, 1, n

先计算in后面的表达式， 返回迭代函数，状态常量和控制变量

将常量和变量作为参数，传入迭代函数

如果返回nil，则停止

否则，将返回的值作为变量继续和状态常量做迭代



通过闭包来解决多状态迭代器

闭包里的local变量还会在上一次的基础上累加

## 8、table

使用table拷贝，两个对象指向同一片内存，当其中一个指向nil

另一个还能访问，类似智能指针。

```Lua
--table的连接
table.concat(xx_table, ",", start, end)
--表，连接符，起始终止索引

--table的插入
table.insert(xx, ind, "")  //在ind处插入对象
//如果ind没写，就默认最后插入、

--table的删除
table.remove(xx, ind) //在ind处删除对象，如果ind没写，默认是最后

--排序
table.sort(xx)


```

## 9、模块和包

目的：另一个文件中，引用另一个文件的模块  same as #include

在对应的模块中，使用table来管理

table.constant = 

function table.func()

然后引入 require("<module>")  //module.lua 

默认去全局的LUA_PATH下找



**如何设定LUA_PATH**  

在~/.bashrc下 添加

```LUA
export LUA_PATH="/home/zxq/xxx/?.lua;;"

然后source ~/.bashrc 刷新
然后将共用的lua模块放到相应文件夹下即可
2
#P asd #C111111 asdfas #n safsasf #C222222 kkkkk#n
```



加载C库中的函数，通过动态链接的方式

指定.so文件

```Lua
local path = "/usr/local/lua/lib/libluasocket.so"
local f = assert(loadlib(path, "luaopen_socket")) //指定路径和函数名
f()  --打开库函数
```

# 3、高级部分

## 1、元表 metatable

目的：对两个table进行操作

### 1、元方法

#### 1、__index

访问作用

```Lua
setmatatable(table, metatable)
meta = getmetatable(table) --返回表的元表， 并且对元表改变后，会影响到表
--[[将table和metatable绑定
本质上来说metatable是一个table， 使用场景是，当两个table中存在类似的部分，就使用metatable绑定，内部存在__index键值对，有两种场景
1、__index映射一个table表，然后访问的使用去表中查找

playMeta = {   --本质是一个table，通过__index管理
    __index = {
        blood = 3,
        attack = 2,
        defense = 4,
        run = function()
            print("run")
        end,
        shoot = function()
            print("shoot")
        end,
    }
}
mytable2 = setmetatable( {}, playMeta)
print(mytable2.blood)
mytable2.shoot()


2、__index映射一个动态函数， __index = function(table, key)   使用时table.key就会传值

playMeta2 = {
    __index = function(mytable, key)
        if key == "blood" then
            return 100
        end
    end
}
--注意当table中存在blood时，会优先调用table中的
--]]
```



#### 2、__newindex

对表更新，当使用一个table不存在的索引时，查找元表中的__newindex方法，调用元方法，进行更新

如果在table中存在，就直接赋值，不调用元方法

```Lua
mymetatable = {}

mytable = setmetatable({key1 = "value"}, {__newindex = mymetatable})

print(mytable.key1)

mytable.newkey = "新值2"
print("mytable.newkey", mymetatable.newkey) --输出新值2
mytable.key1 = "新值1"
print("mytable.key1", mymetatable.key1)   --输出新值1


方法二、调用function
mytable1 = setmetatable({key1 = "value1"}, {
    __newindex = function(mytable1, key, value) --在调用=时，通过__newindex的function赋值
        rawset(mytable1, key, "\""..value.."\"") --注意转义字符引号  使用方式 "\""
    end
})

mytable1.key1 = "new value"
mytable1.key2 = 4

print(mytable1.key1, mytable1.key2)
```





### 2、运算符的重载

使用方式

```Lua
--[[
__add, __sub, __mul, __div, __mod, __concat, __eq, __lt(小于), __le(小于等于)
]]--

比如将两个table的数据拼接起来
table = tabl1 + table2
在其中一个表中，定义元表，并且设置__add方法

function table_maxn(t)  --需要自己实现maxn函数，求出key的最大
    local mn = 0
    for k, v in pairs(t) do
        if (mn < k) then
            mn = k
        end
    end
    return mn
end

mytable = setmetatable({1, 2, 3}, {
    __add = function(mytable, newtable)
        ind = table_maxn(mytable)
        for i = 1, table_maxn(newtable) do
            table.insert(mytable, ind + i, newtable[i])  //将后者的值添加到前者的最大下标后面
        end
        return mytable
    end
})


secondtable = {4, 5, 8}
mytable1 = secondtable + mytable
for k, v in ipairs(mytable1) do
    print(k ,v)
end


--元素的相加

local function add(t1, t2)
    assert(#t1 == #t2)
    local length = #t1
    for i = 1, length do
        t1[i] = t1[i] + t2[i]
    end
    return t1
end

t1 = setmetatable({1, 2, 3}, {__add = add})
t2 = setmetatable({10, 20, 30}, {__add = add})

for k, v in pairs(t1) do
    print(k, "=>", v)
end

for k, v in pairs(t2) do
    print(k, "=>", v)
end

print("---after add----")
t1 = t1 + t2
for i = 1, #t1 do
    print(t1[i])
end


```

### 3、__call元方法

本质就是重载括号运算符

通过table()访问

```Lua
mytable1 = setmetatable({10, 10}, {
    __call = function(mytable, newmytable)
        sum = 0
        for i = 1, table_maxn(mytable) do
            sum = sum + mytable[i]
        end
        for i = 1, table_maxn(newmytable) do
            sum = sum + newmytable[i]
        end
        return sum
    end
}) 
newtable = {10, 20, 30}
print(mytable1(newtable))   --输出两个表的总和
```

### 4、to_string

可以修改表的输入

print(table) 

```Lua
mytable = setmetatable({ 10, 20, 30 }, {
  __tostring = function(mytable)
    sum = 0
    for k, v in pairs(mytable) do
        sum = sum + v
 end
    return "表所有元素的和为 " .. sum
  end
})
print(mytable)
```



# 4、协程

拥有独立的栈，独立的局部变量和指令指针，同时和其他协程共享全局变量和其他大部分东西

和线程的区别: 一个具有多个线程的程序可以同时运行几个线程，而协程需要彼此协同的运行，任何时刻**只能有一个协同程序**进行，并且只有在被人为明确被挂起时才会被挂起



函数

``` Lua
co = coroutine.create(func)  //传入一个函数，创建协程    warp和这个功能一样
coroutine.resume(co) 重启协程，返回true of false(如果不能执行了，就false) 和yiled的值
coroutine.yield(x)  挂起并返回参数x
coroutine.status(co)   返回参数状态，只有运行，挂起和死亡
coroutine.running()  返回正在跑得coroutine，本质是一个线程，返回的是线程号
 
#!/usr/local/bin/lua


function productor()
    local i = 0
    while true do
        i = i + 1
        send(i)
    end
end

function consumer()
    while true do
        local i = receive()  -- 调用receive
        print(i)   --接收到返回值，并输出
    end
end

function receive()
    local status, value = coroutine.resume(newProductor)  --启动协程，调用productor函数，启动send，每次返会一个i，并挂起相应协程
    return value
end

function send(x)
    coroutine.yield(x)  --返回x，然后挂起
end

newProductor = coroutine.create(productor)
consumer()
```



# 5、文件IO

简单模式

```lua
--通过io.x来访问和存储
file = io.open("test.lua", "r")
io.input(file)
print(io.read())  --读取一行
io.close(file)

file = io.open("test.lua", "a")  --以末尾附加的方式打开文件
io.output(file)
io.write("-- tast.lua 文件末尾注释")
io.close(file)
```



完全模式

```Lua
file = io.open("test.lua", "r")
print(file:read()
file:close()

file = io.open("test.lua", "a")
file:write("--test")
file:close()
```

# 6、面向对象

构建一个类

通过function的方式

```Lua
function create(name, id)
    local obj = {name = name, id = id} --成员属性 初始化
    obj.GetName = function(self)
        return self.name
    end    --通过obj表的映射，构建成员方法
    --更加便捷的构造方式
    function obj:SetName(name)
        self.name = name
    end
    function obj:GetName()
        return self.name
    end
    function obj:SetId(id)
        self.id = id
    end
    function obj:GetId()
        return self.id
    end

    return obj
end

local myCreate = create("sam", 001) --创建方式


继承的实现
local function CreateFootballRobot(name, id, position)
    local obj = create(name, id)   --直接调用子类的create，返回一个table
    obj.position = "right back"
    function obj:SetPosition(p)
        self.positon = p
    end
    function obj:GetPosition()
        return self.position
    end
    return obj
end



将成员变量私有化
local function create2(name, id)
    local data = {name = name, id = id}
    local obj = {} 

    function obj:SetName(name)
        data.name = name
    end

    function obj.GetName()
        return data.name
    end

    function obj:SetId(id)
        data.id = id
    end

    function obj:GetId()
        return data.id
    end

    return obj
end
```



7、进阶话题

