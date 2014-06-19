Relevant lines in LAMMPS code for calculation of gamma:

```C++
#define SINERTIA 0.4 moment of inertia prefactor for sphere
inertiaone = SINERTIA * radius[i] * radius[i] * rmass[i];
t_period = force->numeric(FLERR,arg[5]);
gamma2 = tsqrt * sqrt(7.2 * inertiaone * boltz / (t_period * dt * mvv2e)) / ftm2v;
gamma2 /= sqrt(ratio[type[i]]);
```

$\gamma = \dfrac{1}{\sqrt{r_i}} \dfrac{1}{(\mathrm{ftm2v})} \sqrt{\dfrac{7.2 I k_b T}{\tau \mathrm{dt} (\mathrm{mvv2e})}}$

Have determined values of variables by printing them to the screen via `fprintf(screen, ...)`.

- radius = 0.5
- rmass = 1
- boltz = 1. Presumably boltzmann factor in current units.
- t_period = 1. arg[5] is position of damping time. Kept at 1 in overdamped code, because Joakim says so and it makes sense.
- dt = 0.00005
- ftm2v = 1. From https://sites.google.com/site/scienceuprising/code-packages/lammps/a-dissection-of-lammps-classes, unit conversion factor.
- mvv2e = 1. See ftm2v
- tsqrt = 0.489898. Square root of the Langevin temperature
- ratio[type[i]] = 1. Can't be sure but I think the aspect ratio of the sphere-like particle.

Must make a similar equation for the athermal gamma.

$k_b T$ term should not be there since contribution is not from thermal fluctuations.

Replace by input energy?

$\gamma_\mathrm{ath} = \dfrac{1}{\sqrt{r_i}} \dfrac{1}{(\mathrm{ftm2v})} \sqrt{\dfrac{7.2 I E_\mathrm{ath}}{\tau \mathrm{dt} (\mathrm{mvv2e})}}$
