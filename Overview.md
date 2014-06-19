# Analysis of Joakim's supplied script

```
units           lj
atom_style      hybrid sphere dipole
dimension       2

lattice         sq 1.0
region          box block 0.0 150.0 0.0 150.0 -0.25 0.25

read_restart    dipole.conf
reset_timestep  0

#create_box     1 box
#create_atoms   1 random 2865 245634 box

# need both mass settings due to hybrid atom style
# epsilon = sigma = m = 1 in reduced units

mass            1 1.0
set             group all mass 1.0
set             group all diameter 1.0
#set            group all dipole/random 98934 1e-10

#velocity       all create 1.00 87287 mom yes rot no

# shifted LJ potential, with cut-off at potential minimum (2^(1/6)*sigma)
pair_style      lj/cut 1.122462
pair_coeff      * * 1.0 1.0
pair_modify     shift yes mix arithmetic

neighbor        0.5 bin

# use nve/limit for equilibration from random config
fix             1 all nve/sphere update dipole
#fix            thermostat all langevin 0.6 0.6 1.0 35435847 omega yes zero yes
fix             thermostat all langevin 1.0 1.0 1.0 35435847 omega yes zero yes
#fix            1 all nve/limit 0.1

timestep        0.00005

# add self-propulsion force in direction of dipole vector
#compute        myMu all property/atom mux muy muz mu
#variable       force equal 24.0
#variable       force_x atom v_force*c_myMu[1]/c_myMu[4]
#variable       force_y atom v_force*c_myMu[2]/c_myMu[4]
#variable       force_z atom v_force*c_myMu[3]/c_myMu[4]
#fix            myForce all addforce v_force_x v_force_y v_force_z

# compute "temperature" of passive particles
#compute        myTemp LJ temp/sphere
#fix            Temp all ave/time 100 1 10000 c_myTemp file temp.out mode scalar start 0

# compute rdf's
#compute        myRDF all rdf 300 1 1 1 2 2 2
#fix            RDF all ave/time 100 3000 300000 c_myRDF file rdf.out mode vector start 0

# compute MSD's
compute         MSD_active all msd
variable        MSD_act_tstep equal c_MSD_active[4]
variable        tstep equal step
fix             MSD_act all print 100 "${tstep}  ${MSD_act_tstep}" file msd_active.dat screen no

fix             2d all enforce2d

# thermodynamic output
thermo_style    custom step temp pe press
thermo          1000
thermo_modify   flush yes

# compute deterministic part of velocity
compute         fdet all property/atom f_det_x f_det_y f_det_z

dump            1 all custom 20000 dump.cnf id type x y z mux muy muz c_fdet[1] c_fdet[2] c_fdet[3]

run             10000

write_restart   dipole.conf
```

Joakim recommends I use system size $L=150$, box is $L \times L$.

Have a number of particles from `create_atoms`.

From reduced units, potential width $\sigma=1$, potential depth $\epsilon=1$, mass $m=1$

Particle radius is $R=\sigma / 2 = 1/2$ from `set group all diameter 1`.

Dipole moment is $p=1 \times 10^{-10}$, unused I think.

From Lammps' documentation on fix langevin, "The damp parameter is specified in time units and determines how rapidly the temperature is relaxed. For example, a value of 100.0 means to relax the temperature in a timespan of (roughly) 100 time units". *But* because of overdamped dynamics, relaxation time doesn't play a role and has to be fixed at 1.0.

From self-propulsion line, speed is 24.

Joakim recommends I use $\phi=0.4$ to reproduce clustering transition. From this can work out how many particles are needed,

```python
A_box = L ** 2
A_particle = pi * R_particle ** 2
A_particles = phi * A_box
n_particles = int(round(A_particles / A_particle))
n_particles = 11459
```

Joakim recommends I use Péclet numbers 70, 80, 90, 100; should get clustering at Pe = 90.

From Joakim's paper,

> $\mathrm{Pe} = 3 v_0 \tau_r / \sigma$, where $v_0 = v(0)$ is the propulsion speed of an isolated ABP, $\sigma$ its diameter, and $\tau_r$ its orientational relaxation time".

Joakim says he varied the langevin temperature. He says in practice $\mathrm{Pe}=24 / T$. This means:

- Pe = 70.0, T = 0.34
- Pe = 80.0, T = 0.30
- Pe = 90.0, T = 0.27
- Pe = 100.0, T = 0.24

# Determination of diffusivity

Assuming some diffusivity-type relation, then the mean-squared displacement, $s=d^2$, obeys,

$s = \Gamma t^\alpha$.

To determine $\alpha$,

$\log{s} = \log{\Gamma t^\alpha}$

$\log{s} = \log{\Gamma} + \log{t^\alpha}$

$\log{s} = \log{\Gamma} + \alpha \log{t}$

Fit a straight line to $\log{s}$ against $\log{t}$, have gradient $\alpha$ and intercept $\log{\Gamma}$
