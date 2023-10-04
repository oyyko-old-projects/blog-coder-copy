---
title: "Mogan_note4"
date: 2023-09-03T20:33:01+08:00
tags: []
toc: true
math: false
---
## 如何画曲线
```
public:
  inline curve_rep () {}
  inline virtual ~curve_rep () {}

  inline virtual int nr_components () { return 1; }
  // the number of components of the curve is useful for getting
  // nice parameterizations when concatenating curves

  virtual point evaluate (double t)= 0;
  // gives a point on the curve for its intrinsic parameterization
  // curves are parameterized from 0.0 to 1.0

  array<point> rectify (double eps);
  // returns a rectification of the curve, which, modulo reparameterization
  // has a uniform distance of at most 'eps' to the original curve

  virtual void rectify_cumul (array<point>& a, double eps)= 0;
  // add rectification of the curve  (except for the starting point)
  // to an existing polysegment

  /*
  NOTE: more routines should be added later so that one
  can reliably compute the intersections between curves
  One might for instance take the following:
  */
  virtual double bound (double t, double eps)= 0;
  // return delta such that |t' - t| < delta => |c(t') - c(t)| < eps.

  virtual point grad (double t, bool& error)= 0;
  // compute the first derivative at t.
  // set error= true if this derivative does not exist.

  virtual double curvature (double t1, double t2)= 0;
  // compute a bound for the second derivative between t1 and t2.
  /* FIXME: What is computed is *really* a bound for the curvature,
       not for the norm of the second derivative. Make it precise
       what it is that is computed exactly. */
  // return a very large number if such a bound does not exist.

  // returns the number of control points which belong to the curve.
  // these control points are ordered and come first in pts & cips.
  virtual int get_control_points (array<double>& abs, array<point>& pts,
                                  array<path>& cip);

  virtual array<double> find_closest_points (double t1, double t2, point p,
                                             double eps);

  virtual double find_closest_point (double t1, double t2, point p, double eps,
                                     bool& found);
```
理解这段代码并用中文解释

这段代码定义了一个名为curve_rep的类，它是abstract_struct类的子类。以下是对代码的中文解释：

1. curve_rep类具有默认构造函数和虚析构函数。
2. nr_components()函数返回曲线的组件数，对于连接曲线时获取良好的参数化很有用。
3. evaluate(double t)函数根据曲线的内在参数化，在曲线上给出一个点。曲线的参数化范围是从0.0到1.0。
4. rectify(double eps)函数返回曲线的修正，它与原曲线的距离在重新参数化的情况下最多为'eps'。
5. rectify_cumul(array<point>& a, double eps)函数将曲线的修正（除起始点外）添加到现有的多段线中。
6. bound(double t, double eps)函数返回一个delta值，使得当|t' - t| < delta时，|c(t') - c(t)| < eps。它用于计算曲线之间的交点。
7. grad(double t, bool& error)函数计算t处的一阶导数，并设置error为true，如果该导数不存在。
8. curvature(double t1, double t2)函数计算t1和t2之间的二阶导数的界限。如果不存在这样的界限，它将返回一个非常大的数。
9. get_control_points(array<double>& abs, array<point>& pts, array<path>& cip)函数返回属于曲线的控制点的数量。这些控制点在pts和cip中按顺序排列。
10. find_closest_points(double t1, double t2, point p, double eps)函数查找曲线上与给定点p最接近的点，并返回其参数值的数组。
11. find_closest_point(double t1, double t2, point p, double eps, bool& found)函数查找曲线上与给定点p最接近的点，并返回其参数值。如果找到了最接近的点，将设置found为true。

这段代码定义了一个抽象的曲线表示类，提供了计算曲线上点、修正、交点计算、曲率计算等功能的接口。具体的实现将在该类的派生类中完成。
