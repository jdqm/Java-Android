中介者模式主要是将对象之间的复杂的网状关系转为星状关系，将事实类之间的交互任务，放到中介者中进行，进而解耦同事类。基本框架图如下：

![中介者模式](/Resource/mediator.png)

下面举一个简单的例子，同事类中有一个number字段，当同事类A改变了它的number值，有一个要求是，类B的number是类A的100倍，双方谁改变了，都要影响另外一个类。
```
public abstract class AbstractColleague {
    private int number = 1;

    public int getNumber() {
        return number;
    }

    public void setNumber(int number) {
        this.number = number;
    }

    public void setNumber(int number, AbstractMediator am) {
        this.number = number;
    }
}
```
```
public class ConcreteColleagueA extends AbstractColleague {
    @Override
    public void setNumber(int number, AbstractMediator am) {
        super.setNumber(number);
        am.effectB();
    }
}
```
```
public class ConcreteColleagueB extends AbstractColleague {
    @Override
    public void setNumber(int number, AbstractMediator am) {
        super.setNumber(number);
        am.effectA();
    }
}
```
```
public abstract class AbstractMediator {
    protected AbstractColleague colleagueA;
    protected AbstractColleague colleagueB;

    public AbstractMediator(AbstractColleague colleagueA, AbstractColleague colleagueB) {
        this.colleagueA = colleagueA;
        this.colleagueB = colleagueB;
    }

    public abstract void effectA();
    public abstract void effectB();
}
```
```
public class ConcreteMediator extends AbstractMediator {
    public ConcreteMediator(AbstractColleague colleagueA, AbstractColleague colleagueB) {
        super(colleagueA, colleagueB);
    }

    @Override
    public void effectA() {
        colleagueA.setNumber(colleagueB.getNumber() / 100);
    }

    @Override
    public void effectB() {
        colleagueA.setNumber(colleagueA.getNumber() * 100);
    }
}
```
解耦的关键点在于，中介者持有各个同事类对象的引用。