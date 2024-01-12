---
layout: post
title: "Collision💥Notes"
categories: study
author: "Jixiang Zhang"
---

> There are three main approaches used to modeling contact with "rigid" body systems: 1) rigid contact approximated with stiff compliant contact, 2) hybrid models with collision event detection, impulsive reset maps, and continuous (constrained) dynamics between collision events, and 3) rigid contact approximated with time-averaged forces (impulses) in a time-stepping scheme.[^4]

## Collision Detection

* FCL
* libccd
* Bullet
* ODE

## Collision resolution

normal force + friction force

* MuJoCo
* MJX
* Bullet
* Drake
* ODE
* jiminy
* RigidBodyDynamics.jl

<object data="/files/contact_model.pdf" type="application/pdf" width="700px" height="700px">
    <embed src="/files/contact_model.pdf">
        <p>This browser does not support PDFs. Please download the PDF to view it:
            <a href="/files/contact_model.pdf">Download PDF
            </a>.
        </p>
    </embed>
</object>

---

### implementation of jiminy[^9]

```c++
// Extract some proxies
contactOptions_t const & contactOptions_ = engineOptions_->contacts;

// Compute the penetration speed
float64_t const vDepth = vContactInWorld.dot(nGround);

// Compute normal force
float64_t const fextNormal = - std::min(contactOptions_.stiffness * depth +
                                        contactOptions_.damping * vDepth, 0.0);
fextInWorld.noalias() = fextNormal * nGround;

// Compute friction forces
vector3_t const vTangential = vContactInWorld - vDepth * nGround;
float64_t const vRatio = std::min(vTangential.norm() / contactOptions_.transitionVelocity, 1.0);
float64_t const fextTangential = contactOptions_.friction * vRatio * fextNormal;
fextInWorld.noalias() -= fextTangential * vTangential;

// Add blending factor
if (contactOptions_.transitionEps > EPS)
{
    float64_t const blendingFactor = - depth / contactOptions_.transitionEps;
    float64_t const blendingLaw = std::tanh(2.0 * blendingFactor);
    fextInWorld *= blendingLaw;
}
```

### implementation of RigidBodyDynamics.jl[^2]

```julia
function normal_force(model::HuntCrossleyModel, ::Nothing, z, ż)
    zn = z^model.n
    f = model.λ * zn * ż + model.k * zn # (2) in Marhefka, Orin (note: z is penetration, returning repelling force)
end

function friction_force(model::ViscoelasticCoulombModel, state::ViscoelasticCoulombState,
        fnormal, tangential_velocity::FreeVector3D)
    T = typeof(fnormal)
    μ = model.μ
    k = model.k
    b = model.b
    x = convert(FreeVector3D{SVector{3, T}}, state.tangential_displacement)
    v = tangential_velocity

    # compute friction force that would be needed to avoid slip
    fstick = -k * x - b * v

    # limit friction force to lie within friction cone
    fstick_norm² = dot(fstick, fstick)
    fstick_max_norm² = (μ * fnormal)^2
    ftangential = if fstick_norm² > fstick_max_norm²
        fstick * sqrt(fstick_max_norm² / fstick_norm²)
    else
        fstick
    end
end
```

### implementation of MIT Cheetah Software[^1]

```c++
template <typename T>
void ContactSpringDamper<T>::_groundContactWithOffset(T K, T D) {
  for (size_t i = 0; i < CC::_nContact; i++) {
    Vec3<T> v =
        CC::_cp_frame_list[i].transpose() *
        CC::_model->_vGC[CC::_idx_list[i]];  // velocity in plane coordinates
    T z = CC::_cp_penetration_list[i];       // the penetration into the ground
    T zd = v[2];                             // the penetration velocity
    T zr = std::sqrt(std::max(T(0), -z));  // sqrt penetration into the ground,
                                           // or zero if we aren't penetrating
    T normalForce =
        zr *
        (-K * z - D * zd);  // normal force is spring-damper * sqrt(penetration)

    // set output force to zero for now.
    CC::_cp_local_force_list[i][0] = 0;
    CC::_cp_local_force_list[i][1] = 0;
    CC::_cp_local_force_list[i][2] = 0;

    if (normalForce > 0) {
      CC::_cp_local_force_list[i][2] =
          normalForce;  // set the normal force. This is in the plane's
                        // coordinates for now

      // first, assume sticking
      // this means the tangential deformation happens at the speed of the foot.
      deflectionRate[CC::_idx_list[i]] = v.template topLeftCorner<2, 1>();
      Vec2<T> tangentialSpringForce =
          K * zr *
          _tangentialDeflections[CC::_idx_list[i]];  // tangential force due to
                                                     // "spring"
      Vec2<T> tangentialForce =
          -tangentialSpringForce -
          D * zr * deflectionRate[CC::_idx_list[i]];  // add damping to get
                                                      // total tangential

      // check for slipping:
      T slipForce = CC::_cp_mu_list[i] *
                    normalForce;  // maximum force magnitude without slipping
      T tangentialForceMagnitude =
          tangentialForce
              .norm();  // actual force magnitude if we assume sticking
      T r = tangentialForceMagnitude / slipForce;  // ratio of force/max_force

      if (r > 1) {
        // we are slipping.
        tangentialForce =
            tangentialForce / r;  // adjust tangential force to avoid slipping
        deflectionRate[CC::_idx_list[i]] =
            -(tangentialForce + tangentialSpringForce) / (D * zr);
      }
      // set forces
      CC::_cp_local_force_list[i][0] = tangentialForce[0];
      CC::_cp_local_force_list[i][1] = tangentialForce[1];
    }
    // move back into robot frame
    CC::_cp_force_list[CC::_idx_list[i]] =
        CC::_cp_frame_list[i] * CC::_cp_local_force_list[i];
  }
}
```

### implementation of MuJoCo: mj_step() -> mj_forward() -> mj_forwardSkip() -> mj_fwdConstraint() -> mj_solPGS()

### implementation of Drake: **Pressure Field Contact**[^5] [^6]

> There are many ways to model contact between rigid bodies. Drake uses an approach we call “compliant” contact. In compliant contact, nominally rigid bodies are allowed to penetrate slightly, as if the rigid body had a slightly deformable layer, but whose compression has no appreciable effect on the body’s mass properties. The contact force between two deformed bodies is distributed over a contact patch with an uneven pressure distribution over that patch. It is common in robotics to model that as a single point contact or a set of point contacts. Hydroelastic contact instead attempts to approximate the patch and pressure distribution to provide much richer and more realistic contact behavior. For a high-level overview, see this blog post.[^7] Drake implements two models for resolving contact to forces: point contact and hydroelastic contact.[^8]

---

## Integration on SO(3)

<object data="/files/lie-group-methods.pdf" type="application/pdf" width="700px" height="700px">
    <embed src="/files/lie-group-methods.pdf">
        <p>This browser does not support PDFs. Please download the PDF to view it:
            <a href="/files/lie-group-methods.pdf">Download PDF
            </a>.
        </p>
    </embed>
</object>

$$
q_t=q_0+t*v_q \to q_t=q_0*\exp(t*v_q)
$$
[^3]

---

[^1]: Azad, Morteza and Roy Featherstone. “Modeling the contact between a rolling sphere and a compliant ground plane.” (2010).
[^2]: Marhefka, Duane W. and David E. Orin. “A compliant contact model with nonlinear damping for simulation of robotic systems.” IEEE Trans. Syst. Man Cybern. Part A 29 (1999): 566-572.
[^3]: <https://gepettoweb.laas.fr/doc/stack-of-tasks/pinocchio/master/doxygen-html/md_doc_c-maths_se3.html>
[^4]: <https://underactuated.csail.mit.edu/multibody.html#contact>
[^5]: <https://ryanelandt.github.io/projects/pressure_field_contact>
[^6]: <https://github.com/ryanelandt/PressureFieldContact.jl>
[^7]: [Rethinking Contact Simulation for Robot Manipulation](https://medium.com/toyotaresearch/rethinking-contact-simulation-for-robot-manipulation-434a56b5ec88)
[^8]: [Hydroelastic Contact User Guide](https://drake.mit.edu/doxygen_cxx/group__hydroelastic__user__guide.html)
[^9]: <https://github.com/duburcqa/jiminy>
