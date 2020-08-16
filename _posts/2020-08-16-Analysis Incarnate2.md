---
title: 'Analysis Incarnate 2'

---

This is a proof of Euler's Identity using the *Taylor Series*. 

![a](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;e^{i\pi}&plus;1=0)

We can write `e^iphi` as the Taylor Series expansion. 

![b](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;e^{i\phi}=\sum_{n=0}^{\infty}\frac{(i\phi)^{n}}{n!})

Let's visualize this sum. It helps to write out all your terms because you know that the powers of `i` are going to alternate between `1`,`-1`,`i`,and `-i`.

![c](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;=\frac{\left(i\phi&space;\:\right)^0}{0!}&plus;\frac{\left(i\phi&space;\right)^1}{1!}&plus;\frac{\left(i\phi&space;\:\right)^2}{2!}&plus;\frac{\left(i\phi&space;\:\right)^3}{3!}&plus;\frac{\left(i\phi&space;\:\right)^4}{4!}&plus;...)

If we simplify these terms using the powers of `i` rule,  we get : 

![d](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;=\frac{1}{0!}&plus;i\left(\frac{\phi}{1!}\right)-\frac{\phi^2}{2!}-i\left(\frac{\phi^3}{3!}\right)&plus;\frac{\phi^4}{4!}&plus;...)

`(iphi)^0/0!` is equal to `1/0!`, as anything raised to the `0` power is one, and `0` factorial is one. On the next term, we get `iphi` as the numerator, and `1!` as the denominator. On the third term (`iphi`^2)/(`2!`),  `i^2` is just `-1`, so we get `-phi^2` over `2!`. On the fourth term, `i^3` is just `-1*i`, which is just `-i`. This is multiplied by `phi^3`/`3!`. This process goes on infinitely. Since we have a complex number, we can factor out the `i` and compare co-efficients of the front and rear parts. 

![e](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;=\left(\frac{1}{0!}-\frac{\phi&space;^2}{2!}&plus;\frac{\phi&space;^4}{4!}-...\right)&plus;i\left(\frac{\phi&space;^1}{1!}-\frac{\phi&space;^3}{3!}&plus;\frac{\phi&space;^5}{5!}-...\right))

As you can see, the addition/subtraction operators alternate after every term. The front half of this equation is the Taylor Series expansion of `cos phi`. The rear half of this equation is the Taylor Series expansion of `sin phi`. You can take the original sum notation and split it into two partial sums, one with the even part, and one with the odd part.  

![f](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;=cos(\phi)&plus;isin(\phi))

We can see that `e^iphi` is equal to `cos phi` + `i sin phi` (Euler's formula). If you plug in `π` into this formula : 

![g](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;e^{i\pi}=&space;cos(\pi)&plus;i&space;sin&space;(\pi))

![h](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;=&space;-1&plus;0)

`cos(π)` is equal to `-1` and `i sin (π)` is just `0` (think of the unit circle!). This leaves us with the most "beautiful" formula in mathematics :

  																			 ![k](https://latex.codecogs.com/png.latex?\dpi{300}&space;\tiny&space;e^{i\pi}&plus;1=&space;0)