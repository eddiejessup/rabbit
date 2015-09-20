In the original model, the translational and rotational diffusion are coupled together, so that they are controlled by one parameter, $T$.

If we vary only the rotational diffusion constant in response to collisions, this uncouples the two, and implies that rotational diffusion arises partly from a mechanism other than thermal noise.

This implies two ways to proceed:

1. Have a local particle temperature, $T_i$, so that $D$ and $D_r$ are still coupled in the same way.
2. Keep $D$ the same, but have two additive contributions to $D_r$, a thermal term coupled to $T$ and $D$ and an athermal term imposed as an additional parameter.

Option (2) seems like the option to go with for now, as the more physically realistic approach

# Langevin dynamics formulation for an athermal rotational diffusion term

In original model, have rotational Langevin dynamics of:

$\partial_t \theta_i = \sqrt{2 D_r} \Lambda_\theta$

so just constant rotational diffusion. Want to change this to,

$\partial_t \theta_i = \sqrt{2 D_{r, i}} \Lambda_\theta$

Need to choose form of $D_{r, i}$. Idea is to make $D_{r, i}$ increase when collisions occur. Two options come to mind: 

1. Naively, would think of collision as being small inter-particle distance.
2. From conversation with Mike and Joakim, they seem to think looking at particle's instantaneous speed as a better option.

So assuming we're going with option (2), we have to decide the form of $D_{r, i}(v)$. Simplest option (and the one suggested by Mike) is an additional constant term appearing below $v_c$:

$D_{r, i} = D_{r} + D_{r, c} \Theta \, (v_c - v_i)$

# Thoughts on approaching/leaving collisions

If two particles are leaving one another, that isn't a collision, so it shouldn't increase the rotational diffusion. Let's look at what $v_i$ is in different cases:

- Low-density limit, v_i = 1
- A 'head-on' collision, conservative force opposes propulsive force, v_i < 1 (may be negative)
- Two particles are close but moving away from each other. This means $F_cons is in the same direction as p, so v_i > 1.

$v_i = \beta D (\mathbf{F}_i + F_p \mathbf{p}_i) \cdot \mathbf{p}_i$

# Implementation

can get atom properties via `double **mu = atom->mu`;

`atom_vec_sphere` defines v, f, torque in this way.

`atom_vec_dipole` defines q, mu in this way.

mu is dipole moment data, a 4-element array. From how it's used in fix_nve_sphere, section `update dipole`,

```C++
if (mu[i][3] > 0.0) {
  g[0] = mu[i][0] + dtv * (omega[i][1]*mu[i][2]-omega[i][2]*mu[i][1]);
  g[1] = mu[i][1] + dtv * (omega[i][2]*mu[i][0]-omega[i][0]*mu[i][2]);
  g[2] = mu[i][2] + dtv * (omega[i][0]*mu[i][1]-omega[i][1]*mu[i][0]);
  msq = g[0]*g[0] + g[1]*g[1] + g[2]*g[2];
  scale = mu[i][3]/sqrt(msq);
  mu[i][0] = g[0]*scale;
  mu[i][1] = g[1]*scale;
  mu[i][2] = g[2]*scale;
}
```

it seems the last element is the dipole moment magnitude; the first 3 are the cartesian orientations.

Can't be bothered to work out if the orientation vector is unit or magnitude mu. Do it sometime or print it out and find out or just always scale by mu.
