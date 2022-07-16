### 协变在Dart中的应用

在Dart中协变默认用于泛型类型，实际上还有用于另一种场景协变方法参数类型，先通过一个例子来看下:
```
//定义动物基类
class Animal {
    final String color;Animal(this.color);
}
//定义Cat类继承Animal
class Cat extends Animal {
    Cat() : super('black cat');
}
//定义Dog类继承Animal
class Dog extends Animal {
    Dog() : super('white dog');
}
//定义一个装动物的笼子类
class AnimalCage {
    void putAnimal(Animal animal) {
        print('putAnimal: ${animal.color}');
    }
}
//定义一个猫笼类
class CatCage extends AnimalCage {
    @overridevoid putAnimal(Animal animal) {
        //注意: 这里方法参数是Animal类型
        super.putAnimal(animal);
    }
}
//定义一个狗笼类
class DogCage extends AnimalCage {
    @overridevoid putAnimal(Animal animal) {
        //注意: 这里方法参数是Animal类型
        super.putAnimal(animal);
    }
}
```
我们需要去重写putAnimal方法，由于是继承自AnimalCage类，所以方法参数类型是Animal.这会造成什么问题呢? 一起来看下:
```
main() {
    //创建一个猫笼对象
    var catCage = CatCage();
    //然后却可以把一条狗放进去，如果按照设计原理应该猫笼子只能put猫。
    catCage.putAnimal(Dog());//这行静态检查以及运行都可以通过。
    //创建一个狗笼对象
    var dogCage = DogCage();
    //然后却可以把一条猫放进去，如果按照设计原理应该狗笼子只能put狗。
    dogCage.putAnimal(Cat());//这行静态检查以及运行都可以通过。
}
```
其实对于上述的出现问题，我们更希望putAnimal的参数更具体些，为了解决上述问题你需要使用 covariant 协变关键字。
```
//定义一个猫笼类
class CatCage extends AnimalCage {
    @overridevoid putAnimal(covariant Cat animal) {
        //注意: 这里使用covariant协变关键字 表示CatCage对象中的putAnimal方法只接收Cat对象
        super.putAnimal(animal);
    }
}
//定义一个狗笼类
class DogCage extends AnimalCage {
    @overridevoid putAnimal(covariant Dog animal) {
        //注意: 这里使用covariant协变关键字 表示DogCage对象中的putAnimal方法只接收Dog对象
        super.putAnimal(animal);}
}
//调用
main() {
    //创建一个猫笼对象
    var catCage = CatCage();
    catCage.putAnimal(Dog());//这时候这样调用就会报错, 报错信息: Error: The argument type 'Dog' can't be assigned to the parameter type 'Cat'.
}
```
为了进一步验证结论，可以看下这个例子:
```
typedef void PutAnimal(Animal animal);
class TestFunction {
    void putCat(covariant Cat animal) {}//使用covariant协变关键字
    void putDog(Dog animal) {}
    void putAnimal(Animal animal) {}
}
main() {
    var function = TestFunction();
    print(function.putCat is PutAnimal);//true 因为使用协变关键字
    print(function.putDog is PutAnimal);//false
    print(function.putAnimal is PutAnimal);//true 本身就是其子类型
}
```
