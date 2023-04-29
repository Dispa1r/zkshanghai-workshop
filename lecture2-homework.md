# 第2课 课后作业

## 第1题 nBits
* 将二进制数按位获取

```
template Num2Bits(n) {
    signal input in;
    signal output out[n];
    var lc1=0;

    var e2=1;
    for (var i = 0; i<n; i++) {
        out[i] <-- (in >> i) & 1;
        // 这里做约束,约束out[i]必须为0/1
        out[i] * (out[i] -1 ) === 0;
        // check sum
        lc1 += out[i] * e2;
        e2 = e2+e2;
    }

    lc1 === in;
}
```

## 第二题 判零 IsZero
设计思路来源于课上PPT
```
template IsZero() {
    signal input in;
    signal output out;

    signal inv;

    inv <-- in!=0 ? 1/in : 0;

    out <== -in*inv +1;
    in*out === 0;
}
```

## 第三题 相等 IsEqual
判断相减是否为0即可

```
template IsEqual() {
    signal input in[2];
    signal output out;

    component isz = IsZero();

    in[1] - in[0] ==> isz.in;

    isz.out ==> out;
}
```

## 第四题 选择器 Selector
```
template QuinSelector(choices) {
    signal input in[choices];
    signal input index;
    signal output out;
    
    // Ensure that index < choices
    component lessThan = LessThan(4);
    lessThan.in[0] <== index;
    lessThan.in[1] <== choices;
    lessThan.out === 1;

    component calcTotal = CalculateTotal(choices);
    component eqs[choices];

    // For each item, check whether its index equals the input index.
    for (var i = 0; i < choices; i ++) {
        eqs[i] = IsEqual();
        eqs[i].in[0] <== i;
        eqs[i].in[1] <== index;
        // 与常规程序设计不同,这里无法直接return某一个值,只能使用类似开关的方法返回
        // eqs[i].out is 1 if the index matches. As such, at most one input to
        // calcTotal is not 0.
        calcTotal.in[i] <== eqs[i].out * in[i];
    }

    // Returns 0 + 0 + 0 + item
    out <== calcTotal.out;
}
```

## 第五题 判负 IsNegative
```
pragma circom 2.1.4;

include "circomlib/poseidon.circom";
// include "https://github.com/0xPARC/circom-secp256k1/blob/master/circuits/bigint.circom";

template Num2Bits (nBits) {
    signal input in;
    signal output out[nBits];

    var acc = 0;
    for (var i = 0; i < nBits; i++) {
        out[i] <-- (in \ (2 ** i)) % 2;
        (1 - out[i]) * out[i] === 0;
        acc += (2 ** i) * out[i];
    }
    acc === in;
}

template CompConstant(ct) {
    signal input in[254];
    signal output out;

    signal parts[127];
    signal sout;

    var clsb;
    var cmsb;
    var slsb;
    var smsb;

    var sum=0;

    var b = (1 << 128) -1;
    var a = 1;
    var e = 1;
    var i;

    for (i=0;i<127; i++) {
        clsb = (ct >> (i*2)) & 1;
        cmsb = (ct >> (i*2+1)) & 1;
        slsb = in[i*2];
        smsb = in[i*2+1];

        if ((cmsb==0)&&(clsb==0)) {
            parts[i] <== -b*smsb*slsb + b*smsb + b*slsb;
        } else if ((cmsb==0)&&(clsb==1)) {
            parts[i] <== a*smsb*slsb - a*slsb + b*smsb - a*smsb + a;
        } else if ((cmsb==1)&&(clsb==0)) {
            parts[i] <== b*smsb*slsb - a*smsb + a;
        } else {
            parts[i] <== -a*smsb*slsb + a;
        }

        sum = sum + parts[i];

        b = b -e;
        a = a +e;
        e = e*2;
    }

    sout <== sum;

    component num2bits = Num2Bits(135);

    num2bits.in <== sout;

    out <== num2bits.out[127];
}

template IsNegative () {
    signal input in;
    signal output out;

    component n2b = Num2Bits(254);

    n2b.in <== in;

    component cmp = CompConstant(10944121435919637611123202872628637544274182200208017171849102093287904247808);
    
    for (var i = 0; i < 254; i++) {
        n2b.out[i] ==> cmp.in[i];
    }

    out <== cmp.out;
}

component main = IsNegative();
```
不能直接只用LessThan(0)是可能LessThan输入信号的范围

## 第五题
```
template LessThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);

    n2b.in <== in[0]+ (1<<n) - in[1];

    out <== n2b.out[n] - 1;
}
```
