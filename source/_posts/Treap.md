---
title: Treap
date: 2020-12-24 12:41:13
tags: Treap
category: 程式
---

# Treap
<!-- more -->

## POJ3580

```cpp=
#include<iostream>
#include<cstdlib>
#include<algorithm>
#include<ctime>
#include<vector>
#define nullptr NULL
using namespace std;
int N = 999999; //total elements
int R = 0;
vector<int> rd(N);
class Treap{
	// key = idx 
	public:
	int val;
	int pri;
	int size;
	int mn;
	bool rev;
	int add_val;
	Treap *lhs;
	Treap *rhs;
	Treap(){}
	Treap(int x){
		val = x;
		pri = rd[R++];
		lhs = nullptr;
		rhs = nullptr;
		size = 1;
		mn = x;
		rev = false;
		add_val = 0;
	}
};

inline int get_size(Treap *t){
	if(t == nullptr){
		return 0;
	}
	else{
		return t->size;
	}
}
inline void pull(Treap *t){
	if(t == nullptr){
		return;
	}
	t -> size = get_size(t->lhs) + get_size(t->rhs) + 1;
	t -> mn = t -> val;
	if(t->lhs != nullptr){
		t -> mn = min(t->mn, t->lhs->mn);
	}
	if(t->rhs != nullptr){
		t -> mn = min(t->mn, t->rhs->mn);
	}
    return;
}
inline void push(Treap *t){
	if(t == nullptr){
		return;
	}
	if(t->add_val != 0){
		//cout << "a" << endl;
		if(t->lhs != nullptr){
			//cout << "b" << endl;
			t->lhs->val += t->add_val;
			t->lhs->add_val += t->add_val;
			t->lhs->mn += t->add_val;
		}
		if(t->rhs != nullptr){
			//cout << "c" << endl;
			t->rhs->val += t->add_val;
			t->rhs->add_val += t->add_val;
			t->rhs->mn += t->add_val;
		}
		//cout << "d" << endl;
		t->add_val = 0;
	}
	if(t->rev){
		if(t->lhs != nullptr){
			swap(t->lhs->lhs, t->lhs->rhs);
			t->lhs->rev = t->lhs->rev^1;
		}
		if(t->rhs != nullptr){
			swap(t->rhs->rhs, t->rhs->lhs);
			t->rhs->rev = t->rhs->rev^1;
		}
		t->rev = false;
	}
	return;
}
inline void pt(Treap *t){
	if(t == nullptr){
		return;
	}
	push(t);
	pt(t->lhs);
	cout << t->val << ' ';
	pt(t->rhs);
	return;
}
inline int key(Treap *t){
	return get_size(t->lhs)+1;
}
inline Treap* merge(Treap *a, Treap *b){
	if(a == nullptr){
		return b;
	}
	if(b == nullptr){
		return a;
	}
	//cout << "m" << endl;
	if(a->pri > b->pri){
		push(a);
		a->rhs = merge(a->rhs, b);
		pull(a);
		return a;
	}
	else{
		push(b);
		b->lhs = merge(a, b->lhs);
		pull(b);
		return b;
	}
}
inline void split(Treap *t, int k, Treap *&a, Treap *&b){
	if(t == nullptr){
		a = nullptr;
		b = nullptr;
		return;
	}
    push(t);
	if(k < key(t)){
		b = t;
		split(t->lhs, k, a, b->lhs);
		pull(b);
	}
	else{
		a = t;
		split(t->rhs, k-key(t), a->rhs, b);
		pull(a);
	}
	return;
}
inline void add(Treap *&t, int lb, int rb, int add_val){
	Treap *a, *b, *c, *d;
	//cout << "1" << endl;
	split(t, lb-1, a, b);
	//cout << "a: "; pt(a); cout << endl;
	//cout << "b: "; pt(b); cout << endl;
	
	//cout << "2" << endl;
	split(b, rb-lb+1, c, d);
	//cout << "c: "; pt(c); cout << endl;
	//cout << "d: "; pt(d); cout << endl;

	//cout << "3" << endl;
	if (c!=nullptr) {
		c->add_val += add_val;
		c->val += add_val;
		c->mn += add_val;
	}
	t = merge(merge(a, c), d);
	//cout << "4" << endl;
	return;
}
Treap tmp[999999];
int n = 0;
inline Treap* new_Treap(int val){
	tmp[n] = Treap(val);
	return &tmp[n++];
}
inline void insert(Treap *&t, int idx, int val){
	Treap *a, *b;
	split(t, idx, a, b);
	t = merge(merge(a, new_Treap(val)), b);
	return;
}
inline void del(Treap *&t, int idx){
	Treap *a, *b, *c, *d;
	split(t, idx-1, a, b);
	split(b, 1, c, d);
    /*
    cout << "a: ";
    pt(a);
    cout << endl;
    cout << "b: ";
    pt(b);
    cout << endl;
    cout << "c: ";
    pt(c);
    cout << endl;
    cout << "d: ";
    pt(d);
    cout << endl;
    */
	t = merge(a, d);
	return;
}
inline int get_min(Treap *&t, int lb, int rb){
	Treap *a, *b, *c, *d;
	split(t, lb-1, a, b);
	split(b, rb-lb+1, c, d);
	if(c == nullptr){
        t = merge(merge(a, c), d);
		return 0;
	}
	int mn = c->mn;
	t = merge(merge(a, c), d);
	return mn;
}
inline void reverse(Treap *&t, int lb, int rb){
	Treap *a, *b, *c, *d;
	split(t, lb-1, a, b);
	split(b, rb-lb+1, c, d);
    if(c == nullptr){
        t = merge(merge(a, c), d);
        return;
    }
	swap(c->lhs, c->rhs);
	c->rev = c->rev^1;
	t = merge(merge(a, c), d);
	return;
}
inline void revolve(Treap *&t, int lb, int rb, int rt){
	int len = rb - lb + 1;
	rt = rt % len;
	Treap *a, *b, *c, *d;
	split(t,  lb-1, a, b);
	split(b, len-rt, b, c);
	split(c, rt, c, d);
	t = merge(merge(a, c), merge(b, d));
	return;
}


int main(){
	ios_base::sync_with_stdio(0); cin.tie(0);
	for(int i = 0; i < N; i++){
		rd[i] = i;
	}
	random_shuffle(rd.begin(), rd.end());
	int sz;
	int q;
	Treap* root = nullptr;
	cin >> sz;
	while(sz--){
		int val;
		cin >> val;
		root = merge(root, new_Treap(val));
	}
    //pt(root);
    //cout << endl;
	cin >> q;
	while(q--){
		string op;
		cin >> op;
		if(op == "ADD"){
			int lb, rb, add_val;
			cin >> lb >> rb >> add_val;
            //cout << "add: " << lb << ' ' << rb << ' ' << add_val << endl;
			add(root, lb, rb, add_val);
			//pt(root);
			//cout << endl;
		}
		else if(op == "REVERSE"){
			int lb, rb;
			cin >> lb >> rb;
            //cout << "reverse: " << lb << ' ' << rb << endl;
			reverse(root, lb, rb);
			//pt(root);
			//cout << endl;
		}
		else if(op == "REVOLVE"){
			int lb, rb ,rt;
			cin >> lb >> rb >> rt;
            //cout << "revolve: " << lb << ' ' << rb << ' ' << rt << endl;
			revolve(root, lb, rb, rt);
			//pt(root);
			//cout << endl;
		}
		else if(op == "INSERT"){
			int idx, val;
			cin >> idx >> val;
            //cout << "insert: " << idx << ' ' << val << endl;
			insert(root, idx, val);
			//pt(root);
			//cout << endl;
		}
		else if(op == "DELETE"){
			int idx;
			cin >> idx;
            //cout << "delete: " << idx << endl;
			del(root, idx);
			//pt(root);
			//cout << endl;
		}
		else if(op == "MIN"){
			int lb, rb;
			cin >> lb >> rb;
            //cout << "min: " << lb << ' ' << rb << endl;
			cout << get_min(root, lb, rb) << endl;
			//pt(root);
			//cout << endl;
		}
	}
}
```
