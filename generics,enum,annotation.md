JAVA Generics, Enum, Annotation
==================

# 제네릭스란?
* 자바 JDK 1.5 이후에 도입된 개념
* 다양한 타입의 객체들을 다루는 메서드나 클래스에 **컴파일 시의 타입체크**를 해주는 기능  

```
// 일반 클래스 Box
class Box {
	
	Object item;
	
	void setItem(Object item) {
		this.item = item;
	}
	Object getItem() {
		return item;
	}
	
}
```

```
// 제네릭 클래스 Box
class Box<T> { //T는 타입 변수, 타입 변수는 ArrayList<E> 혹은 Map<K,V> 등으로 사용 가능
	
	T item; // Object를 모두 T로 변경
	
	void setItem(T item) {
		this.item = item;
	}
	T getItem() {
		return item;
	}
	
}
```


클래스에 타입 변수를 지정하면 제네릭 클래스가 되고, 제네릭 클래스의 객체를 생성할 때는 타입 T 대신에 사용될 실제 타입을 지정해야함.

```
//Box b = new Box(); 제네릭 클래스이므로 실제 타입을 지정해야함.
Box<String> b = new Box<String>();
//b.setItem(new Object()); String이외의 타입은 지정 불가.
b.setItem("abc");
String item = b.getItem(); // 형변환 필요 없음.
```

# 제네릭스의 장점

- 제네릭스를 사용함으로써 컴파일 시에 '타입 체크'를 진행하기 때문에, 개발자가 의도하지 않은 타입이 들어갔는 지 사전에 알 수 있으며, 이러한 점이 **타입 안정성**을 제공함.  
- 객체를 꺼낼 때 형변환을 생력할 수 있고, 타입체크를 따로 할 필요가 없어 코드가 간결해짐.

# 제네릭스의 제한

```
class Box<T> {
	
	static T item; // static 멤버에 타입 변수 T 사용 불가.
	static int compare (T t1, T t2) {	} // Box의 모든 객체에 대해 동일하게 동작해야함.(static) 
	
	T[] itemArr; // T타입의 배열을 위한 참조변수(가능)
	
	T[] toArray() {
		T[] tmpArr = new T[itemArr.length]; // 제네릭 배열 생성 불가
		
		return tmpArr;
	}
	
}
```
- 제네릭 클래스 Box의 객체를 생성할 때, 객체별로 다른 타입을 지정하는게 맞음.
- 하지만 static 멤버의 경우 모든 객체에 대해 동일하게 동작해야하므로 타입 변수 T를 사용할 수 없음.
- 또한 지네릭 배열 타입의 참조변수를 선언하는 것은 가능하지만, new 연산자로 배열을 생성하는 것은 불가능.(new로 객체를 생성할 때 어떤 타입인 지 모름)

# 제네릭 클래스의 객체 생성과 사용
```
class Box<T> {
	
	ArrayList<T> list = new ArrayList<T>();
	
	void add(T item) { list.add(item); }
	T get(int i) { return list.get(i); }
	ArrayList<T> getList() { return list; }
	int size() { return list.size(); }
	public String toString() { return list.toString(); }
	
}
```
```
Box<Apple> appleBox = new Box<Apple>(); // 가능
Box<Apple> appleBox = new Box<Grape>(); // 에러
```

- Box<T>의 객체에는 T 타입의 객체만 저장할 수 있음.
- ArrayList<T> list의 객체는 Box와 같은 T 타입만 저장할 수 있음.
	
# 제한된 제네릭 클래스


```
public class generics {
	
	public static void main(String[] args) {
		FruitBox<Toy> fruitBox = new FruitBox<Toy>();
		fruitBox.add(new Toy());
	}
}

class FruitBox<T> {
	ArrayList<T> list = new ArrayList<T>();	
}
```

- 클래스 이름이 FruitBox여도 타입 변수를 T로 명시하면 Toy 타입의 객체를 넣을 수 있음
- 클래스 이름대로 과일만 담기 위해서는 타입 변수 T에 제한이 필요함.

```
class FruitBox<T extends Fruit> { // Fruit의 자손만 타입으로 지정가능
	ArrayList<T> list = new ArrayList<T>();	
	void add(T item) { list.add(item); }
	...
}
```

- T extends Fruit 처럼 Fruit 클래스의 자손들만 넣을 수 있도록 제한.
- 멤버변수 list와  메소드 add() 모두 Fruit 클래스의 자손들만 들어갈 수 있음.

```
interface Eatable() {}
class FruitBox<T extends Eatable> { ... }
class FruitBox<T extends Fruit & Eatable> { ... }
```

- 클래스가 아니라 인터페이스를 구현해야 하는 제약이 필요하다면 implements가 아닌 extends 사용.
- 어떤 클래스의 자손이면서 인터페이스도 구현해야 한다면 &로 표현.


# 와일드 카드

```
class Juicer {
	static Juice makeJuice(FruitBox<Fruit> box) {
		String tmp = "";
		for(Fruit f : box.getList()) tmp += f + " ";
		return new Juice (tmp);
	}
}
```

- Juicer 클래스는 제네릭 클래스가 아니며, 제네릭 클래스라고 하더라도 static 메소드에는 타입 매개변수 T를 사용할 수 없음.
- 따라서 아예 제네릭스를 적용하지 않거나 위처럼 타입 매개변수 T 대신 특정 타입을 지정해야 함.

```
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

System.out.println(Juicer.makeJuice(fruitBox));
System.out.println(Juicer.makeJuice(appleBox));
```

- 하지만 메소드의 매개 변수의 제네릭 타입을 `FruitBox<Fruit>`으로 고정해놓으면, `FruitBox<Apple>`타입의 객체는 makeJuice()의 매개변수가 될 수 없음.
- 또한 제네릭 타입이 다른 것만으로는 오버로딩도 성립하지 않으므로 오버로딩도 불가능함.

```
class Juicer {
	static Juice makeJuice(FruitBox<? extends Fruit> box) { // 와일드 카드 추가
		String tmp = "";
		for(Fruit f : box.getList()) tmp += f + " ";
		return new Juice (tmp);
	}
}
```
위처럼 와일드 카드를 사용하면 Fruit뿐만 아니라 `FruitBox<Apple>`, `FruitBox<Grape>`도 매개 변수로 들어갈 수 있음.

# 와일드 카드의 종류

1. <? extends T> 와일드 카드의 상한 제한(upper bound) - T와 그 자손들을 구현한 객체들만 매개변수로 가능
2. <? super T> 와일드 카드의 하한 제한(lower bound) -T와 그 조상들을 구현한 객체들만 매개변수로 가능
3. <?> 제한 없음

# 제네릭 메소드

메소드의 선언부에 제네릭 타입이 선언된 메소드

```
class FruitBox<T> {
	...
	static <T> void sort(List<T> list, Comparator<? super T> c) {
		...
	}
}
```
- FruitBox는 제네릭 클래스이며 타입 매개변수 T를 가짐.
- sort()는 제네릭 메소드이며 타입 매개변수 T를 갖고 이 타입 변수는 FruitBox의 타입 매개변수 T와 다름.


```
class Juicer {
	static Juice makeJuice(FruitBox<? extends Fruit> box) {
		String tmp = "";
		for(Fruit f : box.getList()) tmp += f + " ";
		return new Juice (tmp);
	}

	static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
		String tmp = "";
		for(Fruit f : box.getList()) tmp += f + " ";
		return new Juice (tmp);
	}
}
```
- 와일드 카드를 이용했던 makeJuice 메소드를 제네릭 메소드로 표현할 수 있음.
- 제네릭 메소드를 호출할 때는 아래와 같이 타입 변수에 타입을 대입해야 함.

```
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

System.out.println(Juicer.<Fruit>makeJuice(fruitBox));
System.out.println(Juicer.<Apple>makeJuice(appleBox));
```

==================
### 열거형(enums)

# 열거형이란?

```
class Card {
	static final int CLOVER = 0;
	static final int HEART = 1;
	static final int DIAMOND = 2;
	static final int SPADE = 3;
	
	static final int TWO = 0;
	static final int THREE = 1;
	static final int FOUR = 2;
	
	int kind;
	int num;
	
}

class Card {
	enum Kind	{	CLOVER, HEART, DIAMOND, SPADE	} // 열거형 Kind 정의
	enum Value	{	TWO, THREE, FOUR	} // 열거형 Value 정의
	
	Kind kind; // 타입이 int가 아닌 Kind
	Value value;	
}
```

- 위의 예시처럼 한정된 값만을 가지는 데이터 타입

# 열거형의 장점

```
if(Card.CLOVER == Card.TWO) // true지만 의미 상 false여야 함.
if(Card.Kind.CLOVER == Card.Value.TWO) // 값은 같지만 타입이 다르므로 false
```

- 자바의 열거형은 실제 값이 같아도 타입이 다르면 조건식의 결과가 false.
- 값 뿐만 아니라 타입까지 체크하기 때문에 타입에 안전함.
- 일반 상수의 경우 상수의 값이 바뀌면, 해당 상수를 참조하는 모든 소스를 다시 컴파일해야함.
- 열거형 상수는 기존의 소스를 다시 컴파일하지 않아도 됨.

# 열거형의 정의와 사용

- 정의
`enum 열거형이름 { 상수명1, 상수명2, ...}`
`enum Direction { EAST, SOUTH, WEST, NORTH }`

- 사용
```
class Unit {
	int x, y;	// 유닛의 위치
	Direction dir;	// 열거형을 인스턴스 변수로 선언

	void init() {
		dir = Direction.EAST; // 유닛의 방향을 EAST로 초기화
	}
}
```

- 클래스의 static 변수를 참조하는 것과 동일하게 사용.
- 열거형 상수간의 비교에는 "=="를 사용할 수 있으며, equals() 보다 빠른 비교 연산이 가능.
- `<`,`>`와 같은 비교 연산자는 사용할 수 없고, compareTo()는 사용 가능. (왼쪽이 크면 양수, 오른쪽이 크면 음수)
- switch문의 조건식에도 열거형을 사용할 수 있음.

# java.lang.Enum

```
Direction[] dArr = Direction.values();
		
for(Direction d : dArr)
	System.out.printf("%s = %d%n", d.name(), d.ordinal());
```

- 열거형에 정의된 모든 상수를 출력하려면 Enum 클래스의 values() 메소드를 사용.
- values()는 열거형의 모든 상수를 배열에 담아 반환.

이 외에도 Enum 클래스에는 다음과 같은 메소드가 정의되어 있음.

|메소드|설명|
|:---|:---|
|Class<E> getDeclaringClass()|열거형의 Class 객체를 반환한다.|
|String name()|열거형 상수의 이름을 문자열로 반환한다.|
|int ordinal()|열거형 상수가 정의된 순서를 반환한다.(0 부터 시작)|
|T valueOf(Class<T> enumType, String name)|지정된 열거형에서 name과 일치하는 열거형 상수를 반환한다.|
	


