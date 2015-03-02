---
layout: post
title:  "ConcurrentSkipListMap의 doPut method"
date:   2015-02-15 15:28:00
categories: java
tags: shit
shortUrl: 
---

SkipList 의 구조
---------------- 


	 Head nodes          Index nodes
     +-+    right        +-+                      +-+
     |2|---------------->| |--------------------->| |->null
     +-+                 +-+                      +-+
      | down              |                        |
      v                   v                        v
     +-+            +-+  +-+       +-+            +-+       +-+
     |1|----------->| |->| |------>| |----------->| |------>| |->null
     +-+            +-+  +-+       +-+            +-+       +-+
      v              |    |         |              |         |
     Nodes  next     v    v         v              v         v
     +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+
     | |->|A|->|B|->|C|->|D|->|E|->|F|->|G|->|H|->|I|->|J|->|K|->null
     +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+



* SkipList 의 Node 만 따로 보았을 때는 일반적인 Linked List와 동일하다.	
* Index에 의해 검색및 삽입 과정이 O(log n) 에 수행되게 된다.


doPut에 관하여
---------------- 

* private V doPut(K key, V value, boolean onlyIfAbsent)
* 메인 삽입 메소드 이다.
* 중복 key를 지원하지 않기 때문에, 존재하는 key의 경우 값을 replace 한다.
* onlyIfAbsent : ture 일 때, key가 중복되더라도 값이 replace 되지 않는다.

* Tree의 경우 insert과정에서 리벨런싱이 발생하고, 이 리벨런싱을 하는 도중 접근에 대하여 blocking된다. 
* 반면 ConcurrentSkipListMap의 doPut의 경우 병렬 프로그래밍을 대비하여 여러가지 장치를 대비하고 있다.
* ConcurrentSkipListMap의 코드를 살펴본 결과, doPut과정을 다시 시도한다던가, compare를 여러번 수행하는 등의 오버헤드가 발생하더라도 접근에 대한 blocking이 발생하지 않도록 하고 있는것을 알 수 있었다.
* 또, insertion과 delete 동시 발생 시 conflict 발생 도 효율적으로 방지하고 있었다.

* 리벨런싱이 없는 자료구조는 편향될 가능성이 있다. 편향되게 되면 탐색 시 최악의 상황에 O(n)의 수행시간이 발생할 수 있다.
* Node의 index Level을 확률적으로 생성함으로써, 리벨런싱을 하지 않아도 index level이 편향되지 않도록 하고있다.


특징
---------------- 
ConcurrentSkipListMap의 doPut 과정은 최대한 blocking이 발생하지 않도록 하고 있는것을 볼 수 있다.

* cas(CompareAndSwap)를 사용한 swapping : Atomically 하게 동작하도록 지원한다.
* conflict 를 방지하기 위해 insertion 과정에서 delete 된 node를 여러번 반복해서 검사하고 있다. 
* cas의 실패를 미리 예상하여, blocking 되지 않고 다시 처음부터 시도 되도록 한다.


		위 코드의 node 삽입 과정에서 
			- if (n != b.next) 
				 break;
			- if ((c = cpr(cmp, key, n.key)) > 0) {
					b = n;
					n = f;
					continue;
				}
		등, 직전에 찾은 node위치의 beforeNode와 nextNode를 한번더 검사하는 등의 불필요해 보일수 있는 동작이 보인다.
		이런 과정은 사실, casNext 와 같은 cas 과정에서 false 되어 검사를 하지 않아도 되는 부분일 수 있다.
		모든 cas 와 관련된 메소드를 호출하기 전에 이처럼 검사 과을 거친다.
		그러나 __cas까지 도달하게 되면 접근이 blocking되기 때문에__ 오버헤드가 발생하더라도 여러번 검사를 하여 blocking 되는것을 최대한 막고 있다.
		cas 과정까지 도달하여 blocking 되는 것 보다 하나의 쓰레드만 처음부터 다시 시도하도록 하는것이 더 효율적인 것으로 판단한 것 같다.
	

* 마커노드 


	- 병렬 프로그램에서 동시에 발생하는 insertion과 delete 에 의해 beforeNode - node - nextNode 의 구조가 conflict 발생할 수 있다.
	- 따라서, 삭제되는 노드를 marking 하는 방식으로 insertion 과정에서 실패하도록 만듬으로써 conflict를 방지할 수 있다.
	- MarkNode를 사용하는 이유는 mark bit 를 사용하는것 보다 메모리를 덜 사용할 수 있기 때문이다.





doPut mothod의 과정
---------------- 

doPut 과정은 다음과 같은 과정에 의해 발생한다.

1) Node 위치 찾기<br>
2) Node 생성 or  Replace<br>
3) Index 생성 <br>
4) Index 위치 찾기 & Index Link<br>



findPredecessor method
---------------- 

* 다음은 Node의 위치를 찾는 메소드 이다.
* Comparator<? super K> cmp 의 경우 ConcurrentSkipListMap의 생성자에서 지정될 수 있다.
* 해설을 다음 코드의 주석으로 달아 두었다.

    private Node<K,V> findPredecessor(Object key, Comparator<? super K> cmp) {
        if (key == null)
            throw new NullPointerException(); // don't postpone errors
        for (;;) {
            for (Index<K,V> q = head, r = q.right, d;;) {
				// HeadIndex에서 부터 시작한다.
				// index 의 right 인덱스를 확인한다.
				// right index가 null 이라면 down index로 이동한다.
                if (r != null) {  
                    Node<K,V> n = r.node;
                    K k = n.key; 
					// right index의 Node의 key를 얻어온다.
                    if (n.value == null) { 
					// right index의 Node의 value 가 null 이면 index를 삭제(unlink)한다.
					
						// 병렬 프로그래밍으로 시도 되기 때문에 cas 과정이 실패할 수 있다.
						// 그 경우 다시 처음부터 시도하게 된다.
                        if (!q.unlink(r))
                            break;           // restart 
							
                        r = q.right;         // reread r
						
						// unlink가 정상적으로 동작하게 되면 다음 right index를 검사한다.
                        continue;
                    }
					
					// key > rightIndex.node.key 일때, rightIndex 로 이동한다.
                    if (cpr(cmp, key, k) > 0) {
                        q = r;
                        r = r.right;
                        continue;
                    }
                }
				
				// down Index로 이동하는 부분.
				// 가장 아랫쪽 Index까지 도달했을 때, Index의 node를 반환한다.
                if ((d = q.down) == null)
                    return q.node;
                q = d;
                r = d.right;
            }
        }
    }

	
주석만 따로 정리하면 다음과 같다.

- HeadIndex에서 부터 시작한다.
- index 의 rightIndex(r)를 확인한다.
- index의 right 가 null 이라면 down index(d)로 이동한다.<br>
	- right index의 Node의 key를 얻어온다.<br>
	- right index의 Node의 value 가 null 이면 index를 삭제(unlink)한다.<br>
	- 병렬 프로그래밍으로 시도 되기 때문에 cas 과정이 실패할 수 있다.<br>
	- 그 경우 다시 처음부터 시도하게 된다.<br>
	- unlink가 정상적으로 동작하게 되면 다음 right index를 검사한다.<br>
	
- key > rightIndex.node.key 일때, rightIndex 로 이동한다.<br>
- down Index로 이동한다.<br>
- 가장 아랫쪽 Index까지 도달했을 때, Index의 node를 반환한다.<br>

반환된 노드는 마지막 노드이거나, next 노드의 key가 put 하고자 하는 key 보다 크거나 같은 노드이다.


doPut method
---------------- 


	private V doPut(K key, V value, boolean onlyIfAbsent) {
        Node<K,V> z;             // added node
        if (key == null)
            throw new NullPointerException();
        Comparator<? super K> cmp = comparator;
        outer: for (;;) {		
			
            for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
				// findPredecessor 메소드를 이용하여 before Node를 찾는다.
				// 찾은 Node의 next Node (n)를 검사한다.
				
				// next Node가 null이라면 마지막 노드이므로 바로 next 에 link시키게 된다.
                if (n != null) {
				
                    Object v; int c;
                    Node<K,V> f = n.next;
					
					// 병렬 프로그래밍으로 작업되기 때문에 n != b.next 상태가 될 수 있다. 이럴 경우 처음부터 다시 시도하게 된다.
                    if (n != b.next)               // inconsistent read
                        break;
						
					// 만약 nextNode의 value가 null 이라면 node를 delete 하게 한다.
                    if ((v = n.value) == null) {   // n is deleted
					
						//helpDelete는, f.next가 markerNode가 아닌 경우 markerNode로 일단 변경하고 f.next가 markerNode로인 경우 unlink 시킨다.
                        n.helpDelete(b, f);
                        break;
                    }
					
					// next Node를 검사하는 도중 before Node가 삭제(null or markerNode)되었을 수 있다.
					// before Node가 삭제되었을 때, 처음부터 다시 시도하게 된다.
                    if (b.value == null || v == n) // b is deleted
                        break;
												
					// 병렬 프로그래밍에 의해 next node가 변경(새로운 노드 삽입)되었을 상황에 대비한다.
					// key > next.key 일 경우 다음 노드로 이동한다.
                    if ((c = cpr(cmp, key, n.key)) > 0) {
                        b = n;
                        n = f;
                        continue;
                    }
					
					// key 와 next.key가 같을 경우, onlyIfAbsent == false일 때 값을 대체한다.
					// onlyIfAbsent == true 일 시, 중복된 키가 있다면 insert 하지 않는다.
                    if (c == 0) {
                        if (onlyIfAbsent || n.casValue(v, value)) {
                            @SuppressWarnings("unchecked") V vv = (V)v;
                            return vv;
                        }
						
						//n.casValue() 가 실패 할 시 처음부터 다시 시도한다.
                        break; // restart if lost race to replace value
                    }
                    // else c < 0; fall through
                }
				
				//새로운 노드를 생성한다.
                z = new Node<K,V>(key, value, n);
				//beforeNode 에 link 시킨다.
				//실패시, 처음부터 다시 시도한다.
                if (!b.casNext(n, z))
                    break;         // restart if lost race to append to b
					
				//Node insert가 완료할 시 다음 과정으로 이동한다.
                break outer;
            }
        }

		//랜덤 값을 하나 생성한다.
        int rnd = ThreadLocalRandom.nextSecondarySeed();
        if ((rnd & 0x80000001) == 0) { // test highest and lowest bits
            int level = 1, max;
			
			//나누기 2를 하면서, level을 증가 시킨다.
            while (((rnd >>>= 1) & 1) != 0)
                ++level;
			
            Index<K,V> idx = null;
            HeadIndex<K,V> h = head;
			
			//지정된 level만큼의 계층 Index를 생성한다.
            if (level <= (max = h.level)) {
                for (int i = 1; i <= level; ++i)
                    idx = new Index<K,V>(z, idx, null);
            }
			//새로 생성될 level이 head.level 보다 크다면 새로운 headIndex를 생성하여야 한다.
            else { // try to grow by one level
			
				//head.level 보다 +1 로 level을 조정한다.
                level = max + 1; // hold in array and later pick the one to use
				
				//headIndex에 연결 될 index list를 새로 생성한다.
                @SuppressWarnings("unchecked")Index<K,V>[] idxs =
                    (Index<K,V>[])new Index<?,?>[level+1];
                for (int i = 1; i <= level; ++i)
                    idxs[i] = idx = new Index<K,V>(z, idx, null);
					
				//HeadIndex list를 생성한다.
                for (;;) {
                    h = head;
					//병렬 프로그래밍으로 인해 새로운 headIndex가 생성되었을 수 있다.
					//이 경우 idx 새로운 headIndex가 필요없기 때문에 break; 시키게 된다.
                    int oldLevel = h.level;
                    if (level <= oldLevel) // lost race to add level
                        break;
						
					// 새로운 HeadIndex를 생성한다. 
                    HeadIndex<K,V> newh = h;
                    Node<K,V> oldbase = h.node;
                    for (int j = oldLevel+1; j <= level; ++j)
                        newh = new HeadIndex<K,V>(oldbase, newh, idxs[j], j);
						
					//Head Index를 cas 한다.
					//실패시 headIndex 생성을 다시 시도한다.
                    if (casHead(h, newh)) {
                        h = newh;
                        idx = idxs[level = oldLevel];
                        break;
                    }
                }
            }
            // find insertion points and splice in
			// 새로 생성한 index list를 SkipList에 붙인다.
            splice: for (int insertionLevel = level;;) {
                int j = h.level;
				
				//headIndex(q)에서 부터 시작하며, rightIndex를 검사한다. 새로 생성된 Index List의 top(t)부터 연결하면 된다.
                for (Index<K,V> q = h, r = q.right, t = idx;;) {
                    if (q == null || t == null)
                        break splice;
                    if (r != null) {
                        Node<K,V> n = r.node;
                        // compare before deletion check avoids needing recheck
						// index를 찾아가기 위해 compare를 시도한다.
                        int c = cpr(cmp, key, n.key);
						// rightIndex의 node가 삭제되었을 시, index의 unlink를 시도한다.
                        if (n.value == null) {
                            if (!q.unlink(r))
                                break;
                            r = q.right;
                            continue;
                        }
						
						// 현재 탐색중인 Node의 key > nextIndex.node 이라면 다음 index로 이동한다.
                        if (c > 0) {
                            q = r;
                            r = r.right;
                            continue;
                        }
                    }
					
					//현재 탐색 index level과 newIndex의 level 이 같다면 link를 시도한 후 insertionLevel을 -1 해준다.
					
                    if (j == insertionLevel) {
						//병렬 프로그래밍으로 인해 실패할 수 있따. 실패 시 처음부터 다시 시도한다.
                        if (!q.link(r, t))
                            break; // restart
						
						//현재 사입 노드의 value가 null일 시 삭제를 시도한 후 종료한다.
                        if (t.node.value == null) {
							//노드 삭제를 위해 findNode 함수를 호출하는 듯 하다.
                            findNode(key);
                            break splice;
                        }
						
						//insertionLevel가 0일 시 doPut과정을 완료한다.
						//insertionLevel을 -1 시킨다.
                        if (--insertionLevel == 0)
                            break splice;
                    }
					
					//현재 탐색중인 j 가 j < level 일때 부터 새로 생성된 index도 down으로 이동한다.
                    if (--j >= insertionLevel && j < level)
                        t = t.down;
						
					//현재 탐색 노드를 down으로 이동한다.
                    q = q.down;
                    r = q.right;
                }
            }
        }
        return null;
    }


주석만 따로 정리하면 다음과 같다.

- doPut 메소드를 이용하여 before Node를 찾는다.
- 찾은 Node의 next Node (n)를 검사한다.
- next Node가 null이라면 마지막 노드이므로 바로 next 에 link시키게 된다.<br>
	- 병렬 프로그래밍으로 작업되기 때문에 n != b.next 상태가 될 수 있다. 이럴 경우 처음부터 다시 시도하게 된다.<br>
	- 만약 nextNode의 value가 null 이라면 node를 delete 하게 한다.<br>
	- next Node를 검사하는 도중 before Node가 삭제(null or markerNode)되었을 수 있다.<br>
	- before Node가 삭제되었을 때, 처음부터 다시 시도하게 된다.<br>
	- key 와 next.key가 같을 경우, onlyIfAbsent == false일 때 값을 대체한다.<br>
	- onlyIfAbsent == true 일 시, 중복된 키가 있다면 insert 하지 않는다.<br>
	- 병렬 프로그래밍에 의해 next node가 변경(새로운 노드 삽입)되었을 상황에 대비한다.<br>
	- key > next.key 일 경우 다음 노드로 이동한다.<br>
	- key 와 next.key가 같을 경우, onlyIfAbsent == false일 때 값을 대체한다.<br>
	- onlyIfAbsent == true 일 시, 중복된 키가 있다면 insert 하지 않는다.<br>
	- n.casValue() 가 실패 할 시 처음부터 다시 시도한다.<br>
	- 새로운 노드를 생성한다.<br>
	- beforeNode 에 link 시킨다. 실패시, 처음부터 다시 시도한다.<br>
	- Node insert가 완료할 시 다음 과정으로 이동한다.<br>

- 랜덤 값을 하나 생성한다.
- 나누기 2를 하면서, level을 증가 시킨다.
- head.level > level 이라면, 지정된 level만큼의 계층 Index를 생성한다.
- 새로 생성될 level이 head.level 보다 크다면 새로운 headIndex를 생성하여야 한다.<br>

	- head.level 보다 +1 로 level을 조정한다.<br>
	- headIndex에 연결 될 index list를 새로 생성한다.<br>
	- HeadIndex list를 생성한다.<br>
	- 병렬 프로그래밍으로 인해 새로운 headIndex가 생성되었을 수 있다.<br>
	- 이 경우 idx 새로운 headIndex가 필요없기 때문에 break; 시키게 된다.<br>
	- 새로운 HeadIndex를 생성한다. <br>
	- Head Index를 cas 한다. 실패시, headIndex 생성을 다시 시도한다.<br>

- 새로 생성한 index list를 SkipList에 붙인다.<br>
- headIndex(q)에서 부터 시작하며, rightIndex를 검사한다. 새로 생성된 Index List의 top(t) 연결하면 된다.
- index를 찾아가기 위해 compare를 시도한다.<br>
- 현재 탐색중인 Node의 key > nextIndex.node 이라면 다음 index로 이동한다.
- 현재 탐색 index level과 newIndex의 level 이 같다면 link를 시도한 후 insertionLevel을 -1 해준다.<br>
	- index link가 병렬 프로그래밍으로 인해 실패할 수 있다. 실패 시 처음부터 다시 시도한다.<br>
	- 현재 사입 노드의 value가 null일 시 삭제를 시도한 후 종료한다.<br>
	- insertionLevel가 0일 시 doPut과정을 완료한다.<br>
	- 아니라면 insertionLevel을 -1 시킨다.<br>

- 현재 탐색중인 j 가 j < level 일때 부터 새로 생성된 index도 down으로 이동한다.
- 현재 탐색 노드를 down으로 이동한 후 반복한다.


