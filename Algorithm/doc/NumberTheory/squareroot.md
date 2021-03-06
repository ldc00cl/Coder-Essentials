> 本文从属于笔者的[数据结构与算法](https://github.com/wxyyxc1992/just-coder-handbook/tree/master/DataStructure)系列文章。

# SquareRoot

平方根计算一直是计算系统的常用算法，本文列举出几张简单易懂的平方根算法讲解与实现。其中Java版本的代码参考[这里](https://github.com/wxyyxc1992/just-coder-handbook/blob/master/Algorithm/java/src/main/java/wx/algorithm/numbertheory/SquareRootsTest.java)



## Reference

- [计算平方根的算法](http://www.cnblogs.com/xkfz007/archive/2012/05/15/2502348.html)

- [Wiki-Methods of computing square roots](https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Taylor_series)



# Babylonian:巴比伦算法/牛顿法

巴比伦算法可能算是最早的用于计算$\sqrt{S}$的算法之一，因为其可以用牛顿法导出，因此在很多地方也被成为牛顿法。其核心思想在于为了计算`x`的平方根，可以从某个任意的猜测值`g`开始计算。在真实的运算中，我们往往将`g`直接设置为`x`，不过也可以选择其他任何的正数值。那么其计算的迭代过程为:

1.如果猜测值`g`已经足够接近于正确的平方根，算法结束，函数将`g`作为结果返回。

2.如果猜测值`g`不够精确，那么使用`g`和`x/g`的平均值作为新的猜测值。因为这两个值中的一个小于确切的平方根，另一个则大于确切的平方根，选择平均值有助于你得到一个更接近于正确答案的值。

3.把新的猜测值赋予给变量`g`，重复第一步的判断。



综上所述，其计算公式可以表述为:

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/7/4/index.png)

其实现的参考代码地址为[SquareRoots](https://github.com/wxyyxc1992/just-coder-handbook/blob/master/Algorithm/java/src/main/java/wx/algorithm/numbertheory/SquareRoots.java):

```

public double Babylonian() {

    double g = this.value;

    while (isApproximate(g)) {
        g = (g + this.value / g) / 2;
    }

    return g;

}
```

# 基于泰勒公式的级数逼近

微积分中的泰勒级数可以表示为:

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/7/4/3B62337C-0126-4CAE-89BF-9B788D008240.png)

在这个公式中，符号`a`表示某个常量，记号`f'、f''`和`f'''`表示函数`f`的一阶、二阶和三阶导数，以此类推，这个公式称为泰勒公式，基于这个公式，我们平方根公式的展开式为:

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/7/4/indenjgkx.png)

根据该公式我们可以在一定精度内逼近真实值，不过这个公式仍然存在一个问题，即是公式的收敛问题。在泰勒级数展开中，平方根函数的公式当且仅当参数值位于一个有效范围内时才有效，在该范围内计算趋于收敛。该范围即是收敛半径，当我们对平方根函数用`a=1`进行计算时，泰勒级数公式希望`x`处于范围:$0<x<2$之间。如果`x`在收敛半径之外，则展开式中的项会越来越大，泰勒级数离答案也就越来越远。为了解决该问题，我们可以考虑当待开平方数大于4时以4去除它，最后将得到的数乘以相同次数的2即可。其实现的参考代码地址为[SquareRoots](https://github.com/wxyyxc1992/just-coder-handbook/blob/master/Algorithm/java/src/main/java/wx/algorithm/numbertheory/SquareRoots.java):

```

public double TSqrt() {

    //设置修正系数
    double correction = 1;

    //因为要对原值进行缩小,因此设置临时值
    double tempValue = value;

    while (tempValue >= 2) {
        tempValue = tempValue / 4;
        correction *= 2;
    }

    return this.TSqrtIteration(tempValue) * correction;
}

private double TSqrtIteration(double value) {

    double sum = 0, coffe = 1, factorial = 1, xpower = 1, term = 1;

    int i = 0;

    while (Math.abs(term) > 0.000001) {

        sum += term;

        coffe *= (0.5 - i);

        factorial *= (i + 1);

        xpower *= (value - 1);

        term = coffe * xpower / factorial;

        i++;
    }

    return sum;

}
```

# 平方根倒数速算法

首先接收一个32位带符浮点数，然后将之作为一个32位整数看待，以将其向右进行一次逻辑移位的方式将之取半，并用十六进制“魔术数字”0x5f3759df减之，如此即可得对输入的浮点数的平方根倒数的首次近似值；而后重新将其作为浮点数，以牛顿法反复迭代，以求出更精确的近似值，直至求出符合精确度要求的近似值。在计算浮点数的平方根倒数的同一精度的近似值时，此算法比直接使用浮点数除法要快四倍。此算法最早被认为是由约翰·卡马克所发明，但后来的调查显示，该算法在这之前就于计算机图形学的硬件与软件领域有所应用，如SGI和3dfx就曾在产品中应用此算法。而就现在所知，此算法最早由Gary Tarolli在SGI Indigo的开发中使用。虽说随后的相关研究也提出了一些可能的来源，但至今为止仍未能确切知晓此常数的起源。



其实现的参考代码地址为[SquareRoots](https://github.com/wxyyxc1992/just-coder-handbook/blob/master/Algorithm/java/src/main/java/wx/algorithm/numbertheory/SquareRoots.java):

```



public double FastInverseSquareRoot() {

    double tempValue = value;

    double xhalf = 0.5d * tempValue;

    long i = Double.doubleToLongBits(tempValue);

    i = 0x5fe6ec85e7de30daL - (i >> 1);

    tempValue = Double.longBitsToDouble(i);

    tempValue = tempValue * (1.5d - xhalf * tempValue * tempValue);

    tempValue = this.value * tempValue;

    return tempValue;

}


```

# Comparsion:比较

笔者建立了一个专门的单元测试类来比较上述算法的准确度与性能，代码参考[SquareRootsTest](https://github.com/wxyyxc1992/just-coder-handbook/blob/master/Algorithm/java/src/main/java/wx/algorithm/numbertheory/SquareRootsTest.java)，首先在准确度与稳定性测试方面，这几种算法都能达到较好地稳定性，其中平方根倒数速算法相对而言是较好。

```

@Test
public void testBabylonian() {

    for (int i = 0; i < 10000; i++) {
        Assert.assertEquals(2.166795861438391, squareRoots.Babylonian(), 0.000001);

    }

}

@Test
public void testTSqrt() {

    for (int i = 0; i < 10000; i++) {

        Assert.assertEquals(2.166795861438391, squareRoots.TSqrt(), 0.000001);

    }
}

@Test
public void testFastInverseSquareRoot() {

    for (int i = 0; i < 10000; i++) {

        Assert.assertEquals(2.1667948388864198, squareRoots.FastInverseSquareRoot(), 0.000001);

    }
}
```

而在性能测试方面，级数逼近的性能最差，巴比伦算法次之，平方根倒数速算法最好:

```

@Test
public void benchMark() {

    //巴比伦算法计时器
    long babylonianTimer = 0;

    //级数逼近算法计时器
    long tSqrtTimer = 0;

    //平方根倒数速算法计时器
    long fastInverseSquareRootTimer = 0;

    //随机数生成器
    Random r = new Random();

    for (int i = 0; i < 100000; i++) {

        double value = r.nextDouble() * 1000;

        SquareRoots squareRoots = new SquareRoots(value);

        long start, stop;

        start = System.currentTimeMillis();

        squareRoots.Babylonian();

        babylonianTimer += (System.currentTimeMillis() - start);

        start = System.currentTimeMillis();

        squareRoots.TSqrt();

        tSqrtTimer += (System.currentTimeMillis() - start);

        start = System.currentTimeMillis();

        squareRoots.FastInverseSquareRoot();

        fastInverseSquareRootTimer += (System.currentTimeMillis() - start);

    }


    System.out.println("巴比伦算法:" + babylonianTimer);

    System.out.println("级数逼近算法:" + tSqrtTimer);

    System.out.println("平方根倒数速算法:" + fastInverseSquareRootTimer);


}


/**
结果为:
巴比伦算法:17
级数逼近算法:34
平方根倒数速算法:7
**/
```

![](http://153.3.251.190:11900/squareroot)