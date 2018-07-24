---
title: Spark MLlib 正则化
date: 2016-08-05 23:21:01
tags:
 - Spark
 - Spark MLlib
categories:
 - Spark
---

# 4.1 正则化

正则化是为了约束训练过程，使其向生成简单模型的方向收敛，从而避免过拟合。MLlib目前支持下面的正则化方法：

| 范数     |       作用          |
|----------|---------------------|
| L1       | 促进产生稀疏的权值  |
| L2       |              平滑   |

### MLlib正则化抽象类

```scala
abstract class Updater extends Serializable {
  /**
   * Compute an updated value for weights given the gradient, stepSize, iteration number and
   * regularization parameter. Also returns the regularization value regParam * R(w)
   * computed using the *updated* weights.
   *
   * @param weightsOld - Column matrix of size dx1 where d is the number of features.
   * @param gradient - Column matrix of size dx1 where d is the number of features.
   * @param stepSize - step size across iterations
   * @param iter - Iteration number
   * @param regParam - Regularization parameter
   *
   * @return A tuple of 2 elements. The first element is a column matrix containing updated weights,
   *         and the second element is the regularization value computed using updated weights.
   */
  def compute(
      weightsOld: Vector,
      gradient: Vector,
      stepSize: Double,
      iter: Int,
      regParam: Double): (Vector, Double)
}
```

### L1范式

```scala
@DeveloperApi
class L1Updater extends Updater {
  override def compute(
      weightsOld: Vector,
      gradient: Vector,
      stepSize: Double,
      iter: Int,
      regParam: Double): (Vector, Double) = {
    val thisIterStepSize = stepSize / math.sqrt(iter)
    // Take gradient step
    val brzWeights: BV[Double] = weightsOld.asBreeze.toDenseVector
    brzAxpy(-thisIterStepSize, gradient.asBreeze, brzWeights)
    // Apply proximal operator (soft thresholding)
    val shrinkageVal = regParam * thisIterStepSize
    var i = 0
    val len = brzWeights.length
    while (i < len) {
      val wi = brzWeights(i)
      brzWeights(i) = signum(wi) * max(0.0, abs(wi) - shrinkageVal)
      i += 1
    }

    (Vectors.fromBreeze(brzWeights), brzNorm(brzWeights, 1.0) * regParam)
  }
}
```

### L2范式

```scala
@DeveloperApi
class SquaredL2Updater extends Updater {
  override def compute(
      weightsOld: Vector,
      gradient: Vector,
      stepSize: Double,
      iter: Int,
      regParam: Double): (Vector, Double) = {
    // add up both updates from the gradient of the loss (= step) as well as
    // the gradient of the regularizer (= regParam * weightsOld)
    // w' = w - thisIterStepSize * (gradient + regParam * w)
    // w' = (1 - thisIterStepSize * regParam) * w - thisIterStepSize * gradient
    val thisIterStepSize = stepSize / math.sqrt(iter)
    val brzWeights: BV[Double] = weightsOld.asBreeze.toDenseVector
    brzWeights :*= (1.0 - thisIterStepSize * regParam)
    brzAxpy(-thisIterStepSize, gradient.asBreeze, brzWeights)
    val norm = brzNorm(brzWeights, 2.0)

    (Vectors.fromBreeze(brzWeights), 0.5 * regParam * norm * norm)
  }
}
```

