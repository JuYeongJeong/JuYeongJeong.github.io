---
layout: post
title:  "java8 ConcurrentHashMap 소스 분석"
date:   2015-02-14 20:00:00
categories: jekyll,themes
tags: omg,shit
shortUrl: http://goo.gl/JhfZT9
---
java8 ConcurrentHashMap

<h1>ConcurrentHashMap</h1>
ConcurrentHashMap 주로 동기화를 위해 사용되는 map입니다. 기존의 HashMap은 동기화 문제가 있었고, 이 문제를 해결 하기 위해서 HashTable이 나왔습니다. 
하지만 Hashtable은 동기화에 있어서 메소드 단위라 매우 비 효율적인 문제점이 있었고 자바1.6버전부터는 동기화와 속도적인 측면을 모두 소화 해줄  ConcurrentHashMap이 나왔습니다. 저는 1.8버전에서의 동기화적인 측면을 중점으로  개선사항에 대해서 알아보겠습니다.

<h1>HashTable VS ConcurrentHashMap</h1>
HashTable은 동기화를 하기 위해 아래와 같이 메소드 전체에 Lock이 걸려있다. 이러한 방법한 간편하고 안전하겠지만 많은 사용자가 있을 경우 사용성이 떨어지게 됩니다. HashTable을 참조하는 객체가 많을 수록 선점하기 위해 대기하는 시간이 늘어나게 됩니다.
<br>
EX) HashTable Lock Metode
{% highlight java %}
public synchronized V put(K key, V value);
public synchronized V get(Object key);
public synchronized V remove(Object key);
{% endhighlight %}

ConcurrentHashMap은 메소드 전체가 아닌 내부의 특정 부분을 Lock하여 많은 Thread에서 접근 가능 할 수 있도록 사용성을 증가 시켰습니다.
<br>
EX) JAVA8 ConcurrentHashMap putAll Method
{% highlight java %}
  final V putVal(K key, V value, boolean onlyIfAbsent) {
       ....
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
			....
            else {
                V oldVal = null;
                synchronized (f) {
                   // NODE 추가
                }
               ....
            }
        }
		....
    }
{% endhighlight %}

위의 코드를 보시면 synchronized된 부분을 볼 수 있습니다. ConcurrentHashMap은 메소드 전체가 아닌 실제로 사용될 Node(bucket)에만 locking을 하여 여러 Thread에서 동시 접근이 가능하도록 하였습니다.

<h1>JAVA8 ConcurrentHashMap과 이전 버전과의 차이점</h1>

java8 이전 버전(java7)에서는 segments를 이용하여 locking을 하고 segment별로 HashEntry를 가지고 있는 것을 볼 수 있었습니다.

<h4>EX) java7 ConcurrentHashMap constructor<h4>
{% highlight java %}
  public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {//concurrencyLevel은 원하는 segment의 사이즈 이고 default로는 16Size입니다.
		..........
				
		// 실제로 segment를 만드는 코드
        Segment<K,V> s0 =
            new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                             (HashEntry<K,V>[])new HashEntry[cap]);
        Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
        UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
        this.segments = ss;
    }
{% endhighlight %}
위의 코드를 보면 segment를 size만큼 만들고 그 내부에 hashEntry를 만드는 것을 볼 수 있습니다.

<h4>EX) segment와 table의 관계</h4>
<img src="http://juyeongjeong.github.io/assets/segment.jpg">
<br>
위의 그림처럼 segment내부에 table이 있습니다.


<h4>Ex) segment put method</h4>
{% highlight java %}
 final V put(K key, int hash, V value, boolean onlyIfAbsent) {
            HashEntry<K,V> node = tryLock() ? null :
                scanAndLockForPut(key, hash, value);//segment 객체 자체를 lock을 합니다.
            V oldValue;
            try {//실제로 data를 put 합니다.
                HashEntry<K,V>[] tab = table;
                int index = (tab.length - 1) & hash;
                HashEntry<K,V> first = entryAt(tab, index);
                for (HashEntry<K,V> e = first;;) {
                    if (e != null) {//기존에 node가 존재 할 경우 입니다.
                        K k;
                        if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                            oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                                ++modCount;
                            }
                            break;
                        }
                        e = e.next;
                    }
                    else {//기존에 node가 없는 경우 입니다.
                        if (node != null)
                            node.setNext(first);
                        else
                            node = new HashEntry<K,V>(hash, key, value, first);
                        int c = count + 1;
                        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                            rehash(node);
                        else
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = null;
                        break;
                    }
                }
            } finally {
                unlock();//segment unlock
            }
            return oldValue;
        }
{% endhighlight %}
실제data가 put이 일어나는 segment put메소드를 보면 lock을 하는것을 볼 수 있습니다. 하나의 segment만 lock되기 때문에 다른 segment는 다른 thread에서 사용 가능하다는 것입니다.


JAVA8버전에서는 segment개념과 다소 변경된 것들이 있습니다.
<br>
<h4>EX) java8 ConcurrentHashMap constructor</h4>
{% highlight java %}
   public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
{% endhighlight %}
java7 부분 보다 훨씬 코드량이 줄어 든것을 볼 수 있다. 자바8에서는 segment별로 HashEntry를 가지고 있는 것이 아니라는 것을 볼 수 있다. 사실상 concurrencyLevel은 table 사이즈 역할과 다름없다고 생각하면 됩니다.

<h4>EX) java8 ConcurrentHashMap putVal<h4>
{% highlight java %}
  final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();//table이 없을 경우 table생성
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // 처음으로 생성된 노드는 동기화 할 필요가 없기때문에 하지 않았습니다.
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {//실제로 data가 테이블에 추가 되는 것입니다.
                V oldVal = null;
                synchronized (f) {//특정 노드만 동기화 된것을 볼수있습니다.
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                         .......
                        }
                    }
                }
                .......
            }
        }
    .......
    }

{% endhighlight %}

putVal메소드 내부를 보면 노드(bucket) 하나마다 동기화가 적용 된것을 볼수 있습니다. 기존에 만들어진 노드를 사용 할 경우에만  동기화가 적용 되는 것을 볼 수 있습니다.
결론적으로 큰 차이점은 segment을 각각의 node에 1:1 단위로 맵핑시켜 이전 버전보다 한번에 많은 Thread에서 접근 할 수 있도록 한것으로 분석됩니다.

