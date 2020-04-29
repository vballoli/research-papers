---
layout: post
title: EigenDamage - Structured Pruning in the Kronecker-Factored Eigenbasis
categories: [Model pruning]
tags: [UofToronto, Vector Insitite, NVidia]
---
## Summary
This paper proposes *reparameterization* of a network to better prune iteratively using Knocker-Factored EigenBasis(KFE) and apply Hessian-based methods for structural pruning.

Link: [MLR](http://proceedings.mlr.press/v97/wang19g/wang19g.pdf)
<!--end_excerpt-->

This paper extends nearly 3 decade old methods Optimal Brain Damage(OBD), LeCun et al. and Optimal Brain Surgeon(OBS) Hassibi et al., by looking at the weights in a bayesian perspective.

### Background
1. Laplace approximation: $ log(p(\theta \mid D)) = log(p(\theta^* | D)) - \frac{(\theta - \theta^*)^TH(\theta - \theta^*)}{2} $ (Since local minima is assumed, first-order gradient ~= 0).
2. Optimal Brain Damage: For pruning a weight $\theta_q$ i.e. $\Delat \theta_q = - \theta_q$, the loss is measured by $\Delta L){OBD} = \frac{\theta_q^2 * H_{qq}}{2}$, where H is the hessian.
3. Optimal Brain Surgeon: OBS is a solution for an optimization problem defined as: 
>$min_q {min_{\Delta \theta} \frac{\Delta \theta^T H \Delta \theta}{2} s.t. e^T_q \Delta \theta_q + \theta^*_q = 0}$,
 where e_q decides whether $\Delta \theta_q$ is 0 or 1. The solution arrived to is: 
>$\Delta \theta = - \frac{\theta^*_q}{H^-1_{qq}} H^{-1}e_q$ 
and 
>$\Delta L_{OBS} = \frac{(\theta^*_q)^2}{2*H^-1_{qq}}$
4. Knockered Factored Eigenbasis - This method is used to approximate the Fisher matrix to determine the importance of each weight. It essentially boils down to 
    - Computing expected value of gradient(see paper for explicit proof), by approximating it to a product of
    S and A, where S = product of the gradient matrix and it's transpose and A = product of activation tensor and it's transponse, for both Linear and Convolutional layers.
    - Now $F = S X A$
    - To further reduce the computation, $F = (Q_S X Q_A)(\lambda_S X \lambda_A)(Q_S X Q_A)^T$

### Algorithm
1. Extending OBS and OBD
    - Compute $\Delta L$ based on OBS or OBD, where the Hessian is approximated to S(mentioned above)
    - Compute pth percentile of $\Delta L$
    - Threshold \theta
        * if greater than threshold, $\theta \leftarrow 0$(OBD) or $\theta \leftarrow \theta - \frac{S^{-1}e_i X \theta_i^*}{S^{-1}_{ii}}$(OBS)

2. The KFE algorithm is similar to the the OBD/OBS extension, except:
    - $vec{W} = {Q_S X Q_A}vec{W} = vec{Q_A W' Q_S}, where vec{W'} =  (QS X QA)^T vec{W}$
    - Therefore, $\theta = = W^* . diag(\lambda_A)diag(\lambda_S)^T W^*$
    - pth percentile, update $\theta$

The paper further explains in depth handling various methods in pruning like iterative pruning, pruning depthwise filters, etc., but all of them revolve around the central idea of KFE based decomposition/approximation to determine importance of each weight/filter.

## Related information
```
@article{wang2019eigendamage,
  title={Eigendamage: Structured pruning in the Kronecker-factored eigenbasis},
  author={Wang, Chaoqi and Grosse, Roger and Fidler, Sanja and Zhang, Guodong},
  journal={arXiv preprint arXiv:1905.05934},
  year={2019}
}
```