# wing model

## 一. 模型建立

[波动率模型 Wing Model(知乎, maomao.run)](https://zhuanlan.zhihu.com/p/406510913)

![img](https://pic2.zhimg.com/80/v2-86953ccb1f077349e5d62c4bdeac44e1_720w.webp)

Wing Model是期权交易中常见的一种对波动率进行建模的方法. 它通过调整参数, 将市场中一个系列的期权的隐含波动率拟合到一个曲线上. Wing Model 把隐含波动率曲线分为 6 个区域, 以 ATM Forward（期权对应标的远期价）为中心, 左边区域 1, 2, 3 构成 Put Wing, 右边区域 4, 5, 6 构成 Call Wing. 其中, 区域 1, 6 为常数波动率部分, 区域 3, 4 为抛物线部分, 区域 2, 5 则为过渡部分（其实也是抛物线）. x 轴为期权的行权价（或者对数化行权价）, y 轴为期权波动率.

| 名称                   | 参数   | 描述                                                                                       |
| ---------------------- | ------ | ------------------------------------------------------------------------------------------ |
| atm forward            | atm    | 期权对应合成期货价                                                                         |
| volatility reference   | vr(vc) | 中心点参考波动率                                                                           |
| slope reference        | sr(sc) | 中心点参考斜率，也是 Put Wing3 和 Call Wing4 抛物线共同的一次项系数                        |
| put curvature          | pc     | Put Wing3 抛物线的二次项系数                                                               |
| call curvature         | cc     | Call Wing4 抛物线的二次项系数                                                              |
| down cutoff            | dc     | Put Wing 2和3 交界点 x 值, dc<0                                                            |
| up cutoff              | uc     | Call Wing 4和5 交界点 x 值, uc>0                                                           |
| down smoothing range   | dsm    | Put Wing 用来计算 1 和 2 交界点 x 值的参数                                                 |
| up smoothing range     | usm    | Call Wing 用来计算 5 和 6 交界点 x 值的参数                                                |
| skew swimmingness rate | ssr    | 斜率游离系数, 取值范围 $[0, 100]$ , 在计算合成期货价格时, 该参数用来调节 atm 和 ref 的比例 |
| volatility change rate | vcr    | 波动率变化系数                                                                             |
| slope change rate      | scr    | 斜率变化系数                                                                               |
| reference price        | ref    | ssr<100 时, 需要定义 ref 用来描述 vcr 和 scr 对中心点波动率和中心点斜率的影响              |

其中，***atm已知，dc、dsm、uc、usm一般为经验值，vcr、scr、ssr一般使用默认值0、0、100，vc、sc、pc、cc待拟合**.*

代码中取的几个默认参数是：**dc=-0.2, uc=0.2,** dsm=0.5, usm=0.5.

- 以**50etf**为例，当前2022-12-13 13:12近月合成期货为2.682，那么中间两个区域边界分别为$F e^{d c}=2.196,F e^{u c}=3.276$；
- 以**300etf**为例，当前2022-12-13 13:12近月合成期货为4.012，那么中间两个区域边界分别为$F e^{d c}=3.285,F e^{u c}=4.9$；

如果 **dc=-0.15, uc=0.15** , 50中间区域边界为 [2.31, 3.12], 300中间区域边界为 [3.45, 4.66].

除了上述参数, 还需要一些中间参数以便于表示最终函数. 其中,

$F$ 合成期货价格：
$$
F=(a t m)^{\frac{s s r}{100}}(r e f)^{1-\frac{s s r}{100}}
$$
$v c$ 是中心点波动率:
$$
v c=v r-\frac{v c r * s s r *(a t m-r e f)}{r e f}
$$
$\mathrm{sc}$ 为中心点斜率:
$$
s c=s r-\frac{s c r * s s r *(a t m-r e f)}{r e f}
$$
在常规使用的时候，下列参数一般取默认值：
$$
\begin{aligned}
& v c r=0 \\
& s c r=0 \\
& s s r=100
\end{aligned}
$$
将默认值带入中间参数公式，可得：
$$
\begin{aligned}
F & =a t m \\
v c & =v r \\
s c & =s r
\end{aligned}
$$
因此**我们平常使用，只需要 $a t m , v r , s r , p c , c c , d c , u c , d s m, usm$ 这9个参数就 够了**. 依照参数的定义, 我们可以定义出区域之间的 5 个分隔点的 x 坐标, 从左到右依次为：
$$
F e^{d c(1+d s m)}, F e^{d c}, F, F e^{u c}, F e^{u c(1+u s m)}
$$
把这个五个点对数化，即：
$$
x=\log \left(\frac{X}{F}\right)
$$
其中 $x$ 为原 $x$ 坐标, $F$ 为合成期货价格. 对数化后我们重新定义出 5 个分隔点新的 x 坐标, 从左到右依次为:
$$
d c(1+d s m), d c, 0, u c, u c(1+u s m)
$$

### 函数求解

区域 3 和区域 4 的抛物线函数是由参数确定的:
$$
\begin{aligned}
& f_3(x)=p c * x^2+s c * x+v c \quad x \in(d c, 0] \\
& f_4(x)=c c * x^2+s c * x+v c \quad x \in(0, u c]
\end{aligned}
$$
根据各区域连接处导数一致列方程即可求出其他区域的表达式：
$$
\begin{aligned}
& f(x) \\
& = \begin{cases}v c+d c *(2+d s m) * \frac{s c}{2}+(1+d s m) * p c * d c^2 & x \in(-\infty, d c(1+d s m)] \\
-\left(\frac{p c}{d s m}+\frac{s c}{2 * d c * d s m}\right) x^2+\left(1+\frac{1}{d s m}\right)(2 * p c * d c+s c) x+v c-\left(1+\frac{1}{d s m}\right) p c * d c^2 & x \in(d c(1+d s m), d c] \\
\qquad\qquad\qquad\qquad\qquad-sc * \frac{dc}{2*dsm}\\
p c * x^2+s c * x+v c & x \in(d c, 0] \\
c c * x^2+s c * x+v c & x \in(0, u c] \\
-\left(\frac{c c}{u s m}+\frac{s c}{2 * u c * u s m}\right) x^2+\left(1+\frac{1}{u s m}\right)(2 * c c * u c+s c) x+v c-\left(1+\frac{1}{u s m}\right) c c * u c^2 & x \in(u c, u c(1+u s m)] \\
\qquad\qquad\qquad\qquad\qquad-sc * \frac{uc}{2*usm}\\
v c+u c *(2+u s m) * \frac{s c}{2}+(1+u s m) * c c * u c^2 & x \in(u c(1+u s m),+\infty)\end{cases} \\
&
\end{aligned}
$$

## 二. 根据无套利进行推导

[Wing-Model Volatility Skew Manager, 子非鱼](https://dybeta2021.github.io/2021/05/24/wing-model/)根据Jim Gatheral 在[Arbitrage-free SVI volatility surfaces ](extension://nhppiemcomgngbgdeffdgkhnkjlgpcdi/data/pdf.js/web/viewer.html?file=https%3A%2F%2Farxiv.org%2Fpdf%2F1204.0646.pdf)提到的无套利观点和算法对 wing-model 进行公式推导分析，在拟合的基础上进一步根据按定义域分 6 块给出 6 个约束条件判断拟合曲线无蝶式套利.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2020/5/22 11:12 PM
# @Author  : 稻草人
# @contact : dybeta2021@163.com
# @File    : wing_model.py
# @Desc    : orc wing model
from functools import partial

from numpy import ndarray, array, arange, zeros, ones, argmin, minimum, maximum, clip
from numpy.linalg import norm
from numpy.random import normal
from scipy.interpolate import interp1d
from scipy.optimize import minimize


class WingModel(object):

    def skew(moneyness: ndarray, vc: float, sc: float, pc: float, cc: float, dc: float, uc: float, dsm: float,
             usm: float) -> ndarray:
        """

        :param moneyness: converted strike, moneyness
        :param vc:
        :param sc:
        :param pc:
        :param cc:
        :param dc:
        :param uc:
        :param dsm:
        :param usm:
        :return:
        """
        assert -1 < dc < 0
        assert dsm > 0
        assert 1 > uc > 0
        assert usm > 0
        assert 1e-6 < vc < 10  # 数值优化过程稳定
        assert -1e6 < sc < 1e6
        assert dc * (1 + dsm) <= dc <= 0 <= uc <= uc * (1 + usm)

        # volatility at this converted strike, vol(x) is then calculated as follows:
        vol_list = []
        for x in moneyness:
            # volatility at this converted strike, vol(x) is then calculated as follows:
            if x < dc * (1 + dsm):
                vol = vc + dc * (2 + dsm) * (sc / 2) + (1 + dsm) * pc * pow(dc, 2)
            elif dc * (1 + dsm) < x <= dc:
                vol = vc - (1 + 1 / dsm) * pc * pow(dc, 2) - sc * dc / (2 * dsm) + (1 + 1 / dsm) * (
                        2 * pc * dc + sc) * x - (pc / dsm + sc / (2 * dc * dsm)) * pow(x, 2)
            elif dc < x <= 0:
                vol = vc + sc * x + pc * pow(x, 2)
            elif 0 < x <= uc:
                vol = vc + sc * x + cc * pow(x, 2)
            elif uc < x <= uc * (1 + usm):
                vol = vc - (1 + 1 / usm) * cc * pow(uc, 2) - sc * uc / (2 * usm) + (1 + 1 / usm) * (
                        2 * cc * uc + sc) * x - (cc / usm + sc / (2 * uc * usm)) * pow(x, 2)
            elif uc * (1 + usm) < x:
                vol = vc + uc * (2 + usm) * (sc / 2) + (1 + usm) * cc * pow(uc, 2)
            else:
                raise ValueError("x value error!")
            vol_list.append(vol)
        return array(vol_list)


    def loss_skew(cls, params: [float, float, float], x: ndarray, iv: ndarray, vega: ndarray, vc: float, dc: float,
                  uc: float, dsm: float, usm: float):
        """

        :param params: sc, pc, cc
        :param x:
        :param iv:
        :param vega:
        :param vc:
        :param dc:
        :param uc:
        :param dsm:
        :param usm:
        :return:
        """
        sc, pc, cc = params
        vega = vega / vega.max()
        value = cls.skew(x, vc, sc, pc, cc, dc, uc, dsm, usm)
        return norm((value - iv) * vega, ord=2, keepdims=False)


    def calibrate_skew(cls, x: ndarray, iv: ndarray, vega: ndarray, dc: float = -0.2, uc: float = 0.2, dsm: float = 0.5,
                       usm: float = 0.5, is_bound_limit: bool = False,
                       epsilon: float = 1e-16, inter: str = "cubic"):
        """

        :param x: moneyness
        :param iv:
        :param vega:
        :param dc:
        :param uc:
        :param dsm:
        :param usm:
        :param is_bound_limit:
        :param epsilon:
        :param inter: cubic inter
        :return:
        """

        vc = interp1d(x, iv, kind=inter, fill_value="extrapolate")([0])[0]

        # init guess for sc, pc, cc
        if is_bound_limit:
            bounds = [(-1e3, 1e3), (-1e3, 1e3), (-1e3, 1e3)]
        else:
            bounds = [(None, None), (None, None), (None, None)]
        initial_guess = normal(size=3)

        args = (x, iv, vega, vc, dc, uc, dsm, usm)
        residual = minimize(cls.loss_skew, initial_guess, args=args, bounds=bounds, tol=epsilon, method="SLSQP")
        assert residual.success
        return residual.x, residual.fun


    def sc(sr: float, scr: float, ssr: float, ref: float, atm: ndarray or float) -> ndarray or float:
        return sr - scr * ssr * ((atm - ref) / ref)


    def loss_scr(cls, x: float, sr: float, ssr: float, ref: float, atm: ndarray, sc: ndarray) -> float:
        return norm(sc - cls.sc(sr, x, ssr, ref, atm), ord=2, keepdims=False)


    def fit_scr(cls, sr: float, ssr: float, ref: float, atm: ndarray, sc: ndarray,
                epsilon: float = 1e-16) -> [float, float]:
        init_value = array([0.01])
        residual = minimize(cls.loss_scr, init_value, args=(sr, ssr, ref, atm, sc), tol=epsilon, method="SLSQP")
        assert residual.success
        return residual.x, residual.fun


    def vc(vr: float, vcr: float, ssr: float, ref: float, atm: ndarray or float) -> ndarray or float:
        return vr - vcr * ssr * ((atm - ref) / ref)


    def loss_vc(cls, x: float, vr: float, ssr: float, ref: float, atm: ndarray, vc: ndarray) -> float:
        return norm(vc - cls.vc(vr, x, ssr, ref, atm), ord=2, keepdims=False)


    def fit_vcr(cls, vr: float, ssr: float, ref: float, atm: ndarray, vc: ndarray,
                epsilon: float = 1e-16) -> [float, float]:
        init_value = array([0.01])
        residual = minimize(cls.loss_vc, init_value, args=(vr, ssr, ref, atm, vc), tol=epsilon, method="SLSQP")
        assert residual.success
        return residual.x, residual.fun


    def wing(cls, x: ndarray, ref: float, atm: float, vr: float, vcr: float, sr: float, scr: float, ssr: float,
             pc: float, cc: float, dc: float, uc: float, dsm: float, usm: float) -> ndarray:
        """
        wing model

        :param x:
        :param ref:
        :param atm:
        :param vr:
        :param vcr:
        :param sr:
        :param scr:
        :param ssr:
        :param pc:
        :param cc:
        :param dc:
        :param uc:
        :param dsm:
        :param usm:
        :return:
        """
        vc = cls.vc(vr, vcr, ssr, ref, atm)
        sc = cls.sc(sr, scr, ssr, ref, atm)
        return cls.skew(x, vc, sc, pc, cc, dc, uc, dsm, usm)


class ArbitrageFreeWingModel(WingModel):

    def calibrate(cls, x: ndarray, iv: ndarray, vega: ndarray, dc: float = -0.2, uc: float = 0.2, dsm: float = 0.5,
                  usm: float = 0.5, is_bound_limit: bool = False, epsilon: float = 1e-16, inter: str = "cubic",
                  level: float = 0, method: str = "SLSQP", epochs: int = None, show_error: bool = False,
                  use_constraints: bool = False) -> ([float, float, float], float):
        """

        :param x:
        :param iv:
        :param vega:
        :param dc:
        :param uc:
        :param dsm:
        :param usm:
        :param is_bound_limit:
        :param epsilon:
        :param inter:
        :param level:
        :param method:
        :param epochs:
        :param show_error:
        :param use_constraints:
        :return:
        """
        vega = clip(vega, 1e-6, 1e6)
        iv = clip(iv, 1e-6, 10)

        # init guess for sc, pc, cc
        if is_bound_limit:
            bounds = [(-1e3, 1e3), (-1e3, 1e3), (-1e3, 1e3)]
        else:
            bounds = [(None, None), (None, None), (None, None)]

        vc = interp1d(x, iv, kind=inter, fill_value="extrapolate")([0])[0]
        constraints = dict(type='ineq', fun=partial(cls.constraints, args=(x, vc, dc, uc, dsm, usm), level=level))
        args = (x, iv, vega, vc, dc, uc, dsm, usm)
        if epochs is None:
            if use_constraints:
                residual = minimize(cls.loss_skew, normal(size=3), args=args, bounds=bounds, constraints=constraints,
                                    tol=epsilon, method=method)
            else:
                residual = minimize(cls.loss_skew, normal(size=3), args=args, bounds=bounds, tol=epsilon, method=method)

            if residual.success:
                sc, pc, cc = residual.x
                arbitrage_free = cls.check_butterfly_arbitrage(sc, pc, cc, dc, dsm, uc, usm, x, vc)
                return residual.x, residual.fun, arbitrage_free
            else:
                epochs = 10
                if show_error:
                    print("calibrate wing-model wrong, use epochs = 10 to find params! params: {}".format(residual.x))

        if epochs is not None:
            params = zeros([epochs, 3])
            loss = ones([epochs, 1])
            for i in range(epochs):
                if use_constraints:
                    residual = minimize(cls.loss_skew, normal(size=3), args=args, bounds=bounds,
                                        constraints=constraints,
                                        tol=epsilon, method="SLSQP")
                else:
                    residual = minimize(cls.loss_skew, normal(size=3), args=args, bounds=bounds, tol=epsilon,
                                        method="SLSQP")
                if not residual.success and show_error:
                    print("calibrate wing-model wrong, wrong @ {} /10! params: {}".format(i, residual.x))
                params[i] = residual.x
                loss[i] = residual.fun
            min_idx = argmin(loss)
            sc, pc, cc = params[min_idx]
            loss = loss[min_idx][0]
            arbitrage_free = cls.check_butterfly_arbitrage(sc, pc, cc, dc, dsm, uc, usm, x, vc)
            return (sc, pc, cc), loss, arbitrage_free


    def constraints(cls, x: [float, float, float], args: [ndarray, float, float, float, float, float],
                    level: float = 0) -> float:
        """蝶式价差无套利约束

        :param x: guess values, sc, pc, cc
        :param args:
        :param level:
        :return:
        """
        sc, pc, cc = x
        moneyness, vc, dc, uc, dsm, usm = args

        if level == 0:
            pass
        elif level == 1:
            moneyness = arange(-1, 1.01, 0.01)
        else:
            moneyness = arange(-1, 1.001, 0.001)

        return cls.check_butterfly_arbitrage(sc, pc, cc, dc, dsm, uc, usm, moneyness, vc)

    """蝶式价差无套利约束条件
    """


    def left_parabolic(sc: float, pc: float, x: float, vc: float) -> float:
        """

        :param sc:
        :param pc:
        :param x:
        :param vc:
        :return:
        """
        return pc - 0.25 * (sc + 2 * pc * x) ** 2 * (0.25 + 1 / (vc + sc * x + pc * x * x)) + (
                1 - 0.5 * x * (sc + 2 * pc * x) / (vc + sc * x + pc * x * x)) ** 2


    def right_parabolic(sc: float, cc: float, x: float, vc: float) -> float:
        """

        :param sc:
        :param cc:
        :param x:
        :param vc:
        :return:
        """
        return cc - 0.25 * (sc + 2 * cc * x) ** 2 * (0.25 + 1 / (vc + sc * x + cc * x * x)) + (
                1 - 0.5 * x * (sc + 2 * cc * x) / (vc + sc * x + cc * x * x)) ** 2


    def left_smoothing_range(sc: float, pc: float, dc: float, dsm: float, x: float, vc: float) -> float:
        a = - pc / dsm - 0.5 * sc / (dc * dsm)

        b1 = -0.25 * ((1 + 1 / dsm) * (2 * dc * pc + sc) - 2 * (pc / dsm + 0.5 * sc / (dc * dsm)) * x) ** 2
        b2 = -dc ** 2 * (1 + 1 / dsm) * pc - 0.5 * dc * sc / dsm + vc + (1 + 1 / dsm) * (2 * dc * pc + sc) * x - (
                pc / dsm + 0.5 * sc / (dc * dsm)) * x ** 2
        b2 = (0.25 + 1 / b2)
        b = b1 * b2

        c1 = x * ((1 + 1 / dsm) * (2 * dc * pc + sc) - 2 * (pc / dsm + 0.5 * sc / (dc * dsm)) * x)
        c2 = 2 * (-dc ** 2 * (1 + 1 / dsm) * pc - 0.5 * dc * sc / dsm + vc + (1 + 1 / dsm) * (2 * dc * pc + sc) * x - (
                pc / dsm + 0.5 * sc / (dc * dsm)) * x ** 2)
        c = (1 - c1 / c2) ** 2
        return a + b + c


    def right_smoothing_range(sc: float, cc: float, uc: float, usm: float, x: float, vc: float) -> float:
        a = - cc / usm - 0.5 * sc / (uc * usm)

        b1 = -0.25 * ((1 + 1 / usm) * (2 * uc * cc + sc) - 2 * (cc / usm + 0.5 * sc / (uc * usm)) * x) ** 2
        b2 = -uc ** 2 * (1 + 1 / usm) * cc - 0.5 * uc * sc / usm + vc + (1 + 1 / usm) * (2 * uc * cc + sc) * x - (
                cc / usm + 0.5 * sc / (uc * usm)) * x ** 2
        b2 = (0.25 + 1 / b2)
        b = b1 * b2

        c1 = x * ((1 + 1 / usm) * (2 * uc * cc + sc) - 2 * (cc / usm + 0.5 * sc / (uc * usm)) * x)
        c2 = 2 * (-uc ** 2 * (1 + 1 / usm) * cc - 0.5 * uc * sc / usm + vc + (1 + 1 / usm) * (2 * uc * cc + sc) * x - (
                cc / usm + 0.5 * sc / (uc * usm)) * x ** 2)
        c = (1 - c1 / c2) ** 2
        return a + b + c


    def left_constant_level() -> float:
        return 1


    def right_constant_level() -> float:
        return 1


    def _check_butterfly_arbitrage(cls, sc: float, pc: float, cc: float, dc: float, dsm: float, uc: float, usm: float,
                                   x: float, vc: float) -> float:
        """检查是否存在蝶式价差套利机会，确保拟合time-slice iv-curve 是无套利（无蝶式价差静态套利）曲线

        :param sc:
        :param pc:
        :param cc:
        :param dc:
        :param dsm:
        :param uc:
        :param usm:
        :param x:
        :param vc:
        :return:
        """
        # if x < dc * (1 + dsm):
        #     return cls.left_constant_level()
        # elif dc * (1 + dsm) < x <= dc:
        #     return cls.left_smoothing_range(sc, pc, dc, dsm, x, vc)
        # elif dc < x <= 0:
        #     return cls.left_parabolic(sc, pc, x, vc)
        # elif 0 < x <= uc:
        #     return cls.right_parabolic(sc, cc, x, vc)
        # elif uc < x <= uc * (1 + usm):
        #     return cls.right_smoothing_range(sc, cc, uc, usm, x, vc)
        # elif uc * (1 + usm) < x:
        #     return cls.right_constant_level()
        # else:
        #     raise ValueError("x value error!")

        if dc < x <= 0:
            return cls.left_parabolic(sc, pc, x, vc)
        elif 0 < x <= uc:
            return cls.right_parabolic(sc, cc, x, vc)
        else:
            return 0


    def check_butterfly_arbitrage(cls, sc: float, pc: float, cc: float, dc: float, dsm: float, uc: float, usm: float,
                                  moneyness: ndarray, vc: float) -> float:
        """

        :param sc:
        :param pc:
        :param cc:
        :param dc:
        :param dsm:
        :param uc:
        :param usm:
        :param moneyness:
        :param vc:
        :return:
        """
        con_arr = []
        for x in moneyness:
            con_arr.append(cls._check_butterfly_arbitrage(sc, pc, cc, dc, dsm, uc, usm, x, vc))
        con_arr = array(con_arr)
        if (con_arr >= 0).all():
            return minimum(con_arr.mean(), 1e-7)
        else:
            return maximum((con_arr[con_arr < 0]).mean(), -1e-7)
```
