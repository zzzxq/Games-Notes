# 1、哈希表实现

![image-20220702231724425](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220702231724425.png)

```c++
#include<iostream>
#include<functional>
#include<vector>
#include<string>

using namespace std;

class Node {
public:
	Node() = default;
	Node(string, int, Node *);
	string get_key();
	int& get_value();  //返回引用，确保[]运算符 可以修改值 h[2] = 1;
	Node* next();
	void set_next(Node*);  //私有变量 使用这个设置next
	void insert_next(Node*); 
	void erase_next();
private:
	string __key;
	int __value;
	Node* __next;
};


class Hash_Table {
public:
	typedef function<int(string)> HASH_FUNC_T;
	Hash_Table(int, HASH_FUNC_T);
	~Hash_Table();
	bool insert(string, int);  //调用私有的__insert， 控制扩容函数
	bool find(string);  //调用私有的__find
	bool erase(string);
	int& operator[](string);  //调用__find,没找到就先插入__insert(key, 0)， 然后返回__find(key)->get_value()
	int capacity();
	void swap(Hash_Table&);

private:
	int data_cnt, size;
	vector<Node> data;
	HASH_FUNC_T hash_func;
	void __expand();
	Node* __insert(string, int);
	Node* __find(string);
};



Node::Node(string key, int value = 0, Node *node = nullptr) 
	:__key(key), __value(value), __next(node) {}

string Node::get_key() { return __key; }
int& Node::get_value() { return __value;  }
Node* Node::next() { return __next; }

void Node::set_next(Node* node)
{
	this->__next = node;
	return;
}
void Node::insert_next(Node* node)
{
	node->set_next(this->next());
	this->set_next(node);
	return;
}
void Node::erase_next()
{
	Node* temp = this->next();
	if (temp == nullptr) return;
	this->set_next(temp->next());
	delete temp;
	return;
}


Hash_Table::Hash_Table(int n, HASH_FUNC_T hash_func)
	: data_cnt(0), size(n), data(n), hash_func(hash_func) {}

Hash_Table::~Hash_Table()
{
	for (int i = 0; i < size; i++) {
		Node* p = &data[i], *temp;
		while (p->next()) {
			temp = p->next();
			p->set_next(temp->next());
			delete temp;
		}
	}
}
bool Hash_Table::insert(string key, int value = 0)
{
	if (data_cnt == 2 * size) __expand();
	return __insert(key, value);
}
bool Hash_Table::find(string key)
{
	return __find(key);
}
bool Hash_Table::erase(string key)
{
	int ind = hash_func(key) % size;
	Node* p = &data[ind];
	while (p->next() && p->next()->get_key() != key) p = p->next();
	if (p->next() == nullptr) return false;
	p->erase_next();
	data_cnt -= 1;
	return true;
}
int& Hash_Table::operator[](string key)
{
	if (!__find(key))
		insert(key, 0);
	return __find(key)->get_value();
}
int Hash_Table::capacity() { return data_cnt; }
void swap(Hash_Table&);



Node* Hash_Table::__insert(string key, int value)
{
	if (__find(key)) return nullptr;
	data_cnt += 1;
	int ind = hash_func(key) % size;
	Node* p = &data[ind];
	Node* node = new Node(key, value);
	p->insert_next(node);
	return node;
}
Node* Hash_Table::__find(string key)
{
	int ind = hash_func(key) % size;
	Node* p = data[ind].next();
	while (p && p->get_key() != key) p = p->next();
	return p;
}

void Hash_Table::swap(Hash_Table& h)
{
	std::swap(this->data_cnt, h.data_cnt);
	std::swap(this->size, h.size);
	std::swap(this->data, h.data);
	std::swap(this->hash_func, h.hash_func);
}

void Hash_Table::__expand()
{
	Hash_Table h(2 * size, hash_func);
	for (int i = 0; i < size; i++) {
		Node* p = data[i].next();
		while (p) {
			h.insert(p->get_key(), p->get_value());
			p = p->next();
		}
	}
	this->swap(h);
}
```

# 2、sort

![image-20220705083947281](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220705083947281.png)

```c++
#include<iostream>
#include<functional>
using namespace std;


class RandomIterator
{
public:
	RandomIterator(int *a) : p(a) {}
	RandomIterator(const RandomIterator& r) {
		p = r.p;
	}
	RandomIterator& operator=(const RandomIterator& r) {
		p = r.p;
		return *this;
	}
	int& operator*() const {
		return *p;
	}
	RandomIterator operator+(int n) const {
		return RandomIterator(p + n);
	}
	RandomIterator operator-(int n) const {
		return RandomIterator(p - n);
	}
	int operator-(RandomIterator& r) {
		return p - r.p;
	}
	RandomIterator& operator++() {
		++p;
		return *this;
	}
	RandomIterator& operator--() {
		--p;
		return *this;
	}
	RandomIterator& operator++(int) {
		RandomIterator temp = *this;
		++p;
		return temp;
	}
	RandomIterator& operator--(int) {
		RandomIterator temp = *this;
		--p;
		return temp;
	}
	bool operator<(const RandomIterator& r) const {
		return p < r.p;
	}
	bool operator>(const RandomIterator& r) const {
		return r < *this;
	}
	bool operator>=(const RandomIterator& r) const {
		return !(*this < r);
	}
	bool operator<=(const RandomIterator& r) const {
		return !(*this > r);
	}
	bool operator==(const RandomIterator& r) const {
		return !(*this > r) && !(*this < r);
	}
	bool operator!=(const RandomIterator& r) const {
		return !(*this == r);
	}
private:
	int* p;
};


const int threthold = 16;
int get_mid(int a, int b, int c) {
	if (a > b) swap(a, b);
	if (a > c) swap(a, c);
	if (b > c) swap(b, c);
	return b;
}

void quick_sort(RandomIterator first, RandomIterator last, function<bool(int, int)> cmp = less<int>()) {
	while (last - first > threthold) {
		RandomIterator x = first, y = last - 1;
		int pivot = get_mid(*x, *y, *(first + (last - first) / 2));
		while (x <= y) {
			while (cmp(*x, pivot)) x++;
			while (cmp(pivot, *y)) y--;
			if (x <= y) {
				swap(*x, *y);
				x++, y--;
			}
		}
		quick_sort(x, last, cmp);
		last = y + 1;
	}
	return;
}


void insertion_sort(RandomIterator first, RandomIterator last, function<bool(int, int)> cmp = less<int>()) {
	RandomIterator ind = first;
	for (RandomIterator j = first + 1; j < last; ++j) {
		if (cmp(*j, *ind)) ind = j;
	}
	for (RandomIterator j = ind; j > first; --j) {
		swap(*j, *(j - 1));
	}
	for (RandomIterator j = first + 1; j < last; ++j) {
		ind = j;
		while (cmp(*ind, *(ind - 1))) {
			swap(*ind, *(ind - 1));
			--ind;
		}
	}
	return;
}


void my_sort(RandomIterator first, RandomIterator last, function<bool(int, int)> cmp = less<int>()) {
	quick_sort(first, last, cmp);
	insertion_sort(first, last, cmp);
	return;
}
```

此外还有 基于归并思想的链表排序

```c++
void merge(ListNode *&p, ListNode* &q) { //一定要用指针的引用，否则里面的修改，对外面无效
    //指针的本质是地址，开辟一个地址存储指向对象的地址，通过->来修改指向对象，如果一个指针指向空，不会影响另一个指针
        ListNode ret, *pr = &ret;
        while (p && q) {
            if (p->val < q->val) {
                pr->next = p;
                pr = pr->next;
                p = p->next;
            } else {
                pr->next = q;
                pr = pr->next;
                q = q->next;
            }
        }
        if (p) {
            pr->next = p;
        } else {
            pr->next = q;
        }
        p = ret.next;
        q = nullptr;
        return ;

    }
    ListNode* sortList(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return head;
        ListNode *counter[64],  *carry;
        int fill = 0;
        while (head) {
            carry = head;
            head = head->next;
            carry->next = nullptr;
            int i = 0;
            for ( ; i < fill && counter[i]; i++) {
                merge(carry, counter[i]);
            }
            swap(carry, counter[i]);
            if (i == fill) {
                fill++;
            } 
        }
        for (int i = 1; i < fill; i++) merge(counter[i], counter[i - 1]);
        return counter[fill - 1];
    }
```







# 3、shared_ptr

包含ptr_data， 存储了 数据T *,计数器 int *

![image-20220705102809691](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220705102809691.png)

```c++
#include<iostream>
using namespace std;


template<typename T>
class ptr_data {
public:
	ptr_data() : ptr(nullptr), cnt(0){}
	ptr_data(T *p) : ptr(p), cnt(1) {}
	ptr_data(nullptr_t) : ptr(nullptr), cnt(0) {}

	void increase_one() {
		this->cnt++;
	}
	void decrease_one() {
		this->cnt--;
		if (cnt == 0) {
			delete ptr;
		}
	}

	T* ptr;
	std::atomic<int> cnt;
};


template<typename T>
class my_shared_ptr 
{
public:
	my_shared_ptr() : p_data(new ptr_data<T>()) {}
	my_shared_ptr(T *ptr) : p_data(new ptr_data<T>(ptr)){}
	my_shared_ptr(nullptr_t) : p_data(new ptr_data<T>(nullptr)) {}
	my_shared_ptr(const my_shared_ptr<T>& p) : p_data(p.p_data){
		this->p_data->increase_one();
	}
	my_shared_ptr& operator=(const my_shared_ptr<T>& p) {
		if (p_data == p.p_data) return *this;
		this->p_data->decrease_one();
		this->p_data = p.p_data;
		p_data->increase_one();
		return *this;
	}

	T& operator*() {
		return *p_data->ptr;
	}
	T* operator->() {
		return p_data->ptr;
	}

	int use_count() const{
		return p_data->cnt.load();
	}
	~my_shared_ptr() {
		p_data->decrease_one();
	}


private:
	ptr_data<T>* p_data;

};


template<class T, class ...Args>
inline my_shared_ptr<T>
my_make_shared(Args&& ...args) {
	return my_shared_ptr<T>(new T(std::forward<Args>(args)...));
}



class A {
public:
	A(int a, int b, int c) : a(a), b(b), c(c){}
	void func() {
		cout << a << " " << b << " " << c << endl;
	}
	~A() {
		cout << "dptr" << endl;
	}
	int a, b, c;
};

int main()
{
	my_shared_ptr<A> p = my_make_shared<A>(1, 2, 3);
	cout << p.use_count() << endl;
	my_shared_ptr<A> p2 = p;
	cout << p.use_count() << endl;
	p2->func();
	p2 = nullptr;
	cout << p2.use_count() << endl;

}
```

# 4、function

实现的功能

- 继承(基类和派生类)
- 多态(虚函数，纯虚函数)
- 模板(可变参模板，模板偏特化)
- 完美转发，重载运算符



基类包括 run 和get_copy 纯虚函数， 同时析构函数一定要是虚函数

派生类包括normal_function，用于接受函数指针

functor用于接受仿函数， 其中需要增加一个typename CLASS_T， 用于接受不同的类



function 成员对象只有一个基类的指针，可以用于指向不同的可调用对象

生成两个构造函数，一个用于接受函数指针，一个接受类， 分别调用基类指针去接受这两个(多态)

需要重载运算符，实现调用基类指针ptr实现相应派生类的run

需要重载赋值运算符，用于实现拷贝，首先要将原本指向的析构掉，然后通过基类指针调用派生类的Get_copy , 其使用系统默认拷贝构造实现

```c++
#include<iostream>
//#include<functional>

template<typename T, typename ...ARGS>
class Base
{
public:
	virtual T run(ARGS ...args) = 0;
	virtual Base<T, ARGS...>* get_copy() = 0;
	virtual ~Base(){}
};

template<typename T, typename ...ARGS>
class normal_function : public Base<T, ARGS...>
{
public:
	normal_function(T (*ptr)(ARGS...)) : ptr(ptr) {}
	T run(ARGS... args) override{
		return ptr(std::forward<ARGS>(args)...);
	}
	Base<T, ARGS...>* get_copy() override {
		return new normal_function(*this);
	}
private:
	T(*ptr)(ARGS...);
};

template<typename CLASS_T, typename T, typename ...ARGS>
class functor : public Base<T, ARGS...>
{
public:
	functor(CLASS_T obj) : obj(obj) {}
	T run(ARGS ...args) {
		return obj(std::forward<ARGS>(args)...);
	}
	Base<T, ARGS...>* get_copy() override {
		return new functor(*this);
	}
private:
	CLASS_T obj;
};




template<typename T, typename ...ARGS> class function{};

template<typename T, typename ...ARGS>
class function<T(ARGS...)>  //模板偏特化
{
public:
	function(T (*ptr)(ARGS...)) : ptr(new normal_function<T, ARGS...>(ptr)) {}
	template<typename CLASS_T>
	function(CLASS_T obj) : ptr(new functor<CLASS_T, T, ARGS...>(obj)) {}

	T operator()(ARGS ...args) {
		return ptr->run(std::forward<ARGS>(args)...);
	}
	function operator=(function<T(ARGS...)> obj) {
		delete this->ptr;
		this->ptr = obj.ptr->get_copy();
		return *this;
	}

private:
	Base<T, ARGS...>* ptr;
};




void func(int a, int b) {
	std::cout << "normal function " << a + b << std::endl;
	
}

class CMP {
public:
	int a;
	CMP(int a) : a(a) {}
	void operator()(int k1, int k2) {
		std::cout << "functor " << a * (k1 + k2) << std::endl;
	}
};

int main()
{
	CMP cmp(2);
	function<void(int, int)> f1 = func;
	function<void(int, int)> f2 = cmp;
	f1(3, 4);
	f2(3, 4);
	function<int(int, int)> f3 = [](int a, int b) { 
		std::cout << "lambda " << a * b << std::endl;
		return a * b; 
	};
	f3(3, 4);
	return 0;
}
```

# 5、线程池 Thread_pool

实现的功能

- 实现Task类，基于可变参模板， function和bind
- 线程池功能

​    手动start, stop

​    往队列中add_task(模板，确定某个函数)， 以及从队列中get_task

​			使用unique_lock<mutex> lock(m_mutex) 

​			cond.notify_one()  和 cond.wait(lock)， 虚假唤醒

  通过工人执行任务，每个工人不断从线程池里取任务

​			通过unordered_map，映射线程id 和运行状态，只要是true不断取任务，通过改变状态 终止任务



主要思想：需要一个任务队列，里面存放若干任务(自定义一个类，通过function接受任务，通过可变参模板来接受任务参数，通过bind来绑定任务)，定义若干线程，不断执行任务worker，从队列中去取任务， 这里需要使用互斥锁mutex， 以及条件变量condition，互斥的访问，每个线程不断的取任务，通过哈希表映射线程id和工作状态，当 执行stop时，将每个线程的工作状态变成false，才停止执行任务。

```c++
#include<iostream>
#include<unordered_map>
#include<mutex>
#include<functional>
#include<thread>
#include<vector>
#include<queue>
#include<condition_variable>

class Task {
public:
	template<typename FUNC_T, typename ...ARGS>
	Task(FUNC_T func, ARGS ...args) {
		this->func = std::bind(func, std::forward<ARGS>(args)...);
	}
	void run() {
		this->func();
		return;
	}
private:
	std::function<void()> func;
};


class ThreadPool 
{
public:
	ThreadPool(int n = 10) : starting(false), thread_size(n), threads(n) {
		this->start();
		
	}
	void start()
	{
		if (starting == true) return;
		for (int i = 0; i < thread_size; ++i) {
			threads[i] = new std::thread(&ThreadPool::worker, this);
		}
		starting = true;
		return;
	}
	void worker() 
	{
		auto id = std::this_thread::get_id();
		running[id] = true;
		while (running[id]) {
			Task* t = get_task();
			t->run();
			delete t;
		}
		return;
	}
	void stop()
	{
		if (starting == false) return;

		for (int i = 0; i < thread_size; ++i) {
			this->add_task(&ThreadPool::stop_running, this);
		}
		for (int i = 0; i < thread_size; ++i) {
			threads[i]->join();
		}
		for (int i = 0; i < thread_size; ++i) {
			delete threads[i];
			threads[i] = nullptr;
		}
		starting = false;
		return;
	}

	template<typename FUNC_T, typename ...ARGS>
	void add_task(FUNC_T func, ARGS ...args)
	{
		std::unique_lock<std::mutex> lock(m_mutex);
		tasks.push(new Task(func, std::forward<ARGS>(args)...));
		m_cond.notify_one();
		lock.unlock();
		return;
	}


	~ThreadPool()
	{
		this->stop();
		while (!tasks.empty()) {
			delete tasks.front();
			tasks.pop();
		}
	}

private:
	bool starting;
	int thread_size;
	std::vector<std::thread*> threads;
	std::queue<Task*> tasks;
	std::mutex m_mutex;
	std::condition_variable m_cond;
	std::unordered_map<decltype(std::this_thread::get_id()), bool> running;

	void stop_running() 
	{
		auto id = std::this_thread::get_id();
		running[id] = false;
		return;
	}

	Task* get_task()
	{
		std::unique_lock<std::mutex> lock(m_mutex);
		while (tasks.empty()) { m_cond.wait(lock); }
		Task* t = tasks.front();
		tasks.pop();
		lock.unlock();
		return t;
	}

};
```



# 6、红黑树插入删除

```c++
Node *insert_maintain(Node *root) {
    if (!has_red_child(root)) return root;
    if (root->left->color == 0 && root->right->color == 0) {
        if (!has_red_child(root->left) && !has_red_child(root->right)) return root;
        root->color = 0;
        root->left->color = root->right->color = 1;
        return root;
    }
    if (root->left->color == 0 && !has_red_child(root->left)) return root;
    if (root->right->color == 0 && !has_red_child(root->right)) return root;
    if (root->left->color == 0) {
        if (root->left->right->color == 0) {
            root->left = left_rotate(root->left);
        }
        root = right_rotate(root);
        root->color = 0;
        root->left->color = root->right->color = 1;
    } else {
        if (root->right->left->color == 0) {
            root->right = right_rotate(root->right);
        }
        root = left_rotate(root);
        root->color = 0;
        root->left->color = root->right->color = 1;
    }
    return root;

}


Node *erase_maintain(Node *root) {
    if (root->left->color < 2 && root->right->color < 2) return root;
    if (has_red_child(root)) {
        root->color = 0;
        int flag = 0;
        if (root->left->color == 0) {
            root = right_rotate(root);
            flag = 0;
            //root->color = 1;
            //root->right = erase_maintain(root->right);
        } else {
            root = left_rotate(root);
            flag = 1;
            //root->color = 1;
            //root->left = erase_maintain(root->left);
        }
        root->color = 1;
        if (flag == 0) root->right = erase_maintain(root->right);
        else root->left = erase_maintain(root->left);
        return root;
    }
    if ((root->left->color == 2 && !has_red_child(root->right)) ||
        (root->right->color == 2 && !has_red_child(root->left))) {
            root->color += 1;
            root->left->color -= 1;
            root->right->color -= 1;
            return root;
        }
    if (root->left->color == 2) {
        root->left->color = 1;
        if (root->right->right->color != 0) {
            root->right->color = 0;
            root->right = right_rotate(root->right);
            root->right->color = 1;
        }
        root->right->color = root->color;
        root = left_rotate(root);
        //root->color = root->left->color;
        root->right->color = root->left->color = 1;
    } else {
        root->right->color = 1;
        if (root->left->left->color != 0) {
            root->left->color = 0;
            root->left = left_rotate(root->left);
            root->left->color = 1;
        }
        root->left->color = root->color;
        root = right_rotate(root);
        //root->color = root->right->color;
        root->left->color = root->right->color = 1;
    }
    return root;
}
```

