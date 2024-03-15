[FSGM Tutorial](https://pytorch.org/tutorials/beginner/fgsm_tutorial.html)
# algo for det-adv-prop :
![Image](https://github.com/Ashuradhipathi/SkillDevWorkshop/blob/main/Screenshot%202024-03-15%20112019.png)

More concretely, we use the FGSM algorithm [10] with
non-targeted attack to explain Det-AdvProp without loss of
generality. Given an input image x and a bounding box b
over an object of class y, FGSM produces the adversarial
examples by a one-step projected gradient descent:
ˆxcls = P(x +  · sign(∇xLcls(x, y; θ))), (3)
ˆxloc = P(x +  · sign(∇xLloc(x, b; θ))), (4)
ˆx = arg max
x∈{ˆxcls,ˆxloc}
Ldet(x, y, b; θ), (5)
where  is the attack strength and P denotes the projection
onto the norm ball {ˆx | kˆx − xk∞ ≤ }. We first generate
two adversarial examples by maximizing the single task loss
and then choose the one that maximizes the total loss of the
detection task. Equation (5) is the max-max rule to keep the
adversarial example (out of two) that maximizes the total
detection loss Ldet. The inner “max” in the max-max rule
refers to the adversarial examples that maximize Lcls and
Lloc, respectively, and the outer “max” indicates we take
the one that maximizes the total detection loss Ldet. The
overall training objective is given below:
min
θ
Ex∼D;y,b∼B(x)Ldet(x, y, b; θ) + Ldet(ˆx, y, b; θ). (6)
Intuitively, this max-max scheme lets the stronger one
survive between the two adversarial examples ˆxcls and ˆxloc.
By separately attacking the classification and localization
branches, we avoid the gradient misalignment problem.
Since we only produce one adversarial example per clean
image, coupled with one auxiliary batchnorm, the training
does not suffer from the excessive regularization problem
mentioned earlier


