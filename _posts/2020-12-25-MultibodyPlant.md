---
layout: post
title: "Drake MultibodyPlant"
categories: study
author: "Jixiang Zhang"
---

MultibodyPlant 表示互连的多体物理系统。`model_instance_name[i]` 表示所有模型实例（model instances），且 `world_model_instance()` 和 `default_model_instance()` 一直都会存在。

MultibodyPlant 提供的 API 接口：
- Ports
  1. get_body_poses_output_port ()
  2. get_body_spatial_velocities_output_port ()
  3. get_body_spatial_accelerations_output_port ()
  4. get_actuation_input_port (ModelInstanceIndex model_instance)
  5. get_actuation_input_port ()
  6. get_applied_generalized_force_input_port ()
  7. get_applied_spatial_force_input_port ()
  8. get_geometry_query_input_port ()
  9. get_state_output_port ()
  10. get_state_output_port (ModelInstanceIndex model_instance)
  11. get_generalized_acceleration_output_port ()
  12. get_generalized_acceleration_output_port (ModelInstanceIndex model_instance)
  13. get_generalized_contact_forces_output_port (ModelInstanceIndex model_instance)
  14. get_reaction_forces_output_port ()
  15. get_contact_results_output_port ()
  16. get_geometry_poses_output_port ()
- Construction
  1. AddRigidBody (const std::string &name, ModelInstanceIndex model_instance, const SpatialInertia< double > &M_BBo_B)
  2. AddRigidBody (const std::string &name, const SpatialInertia< double > &M_BBo_B)
  3. AddFrame (std::unique_ptr< FrameType< T >> frame)
  4. AddJoint (std::unique_ptr< JointType< T >> joint)
  5. WeldFrames
  6. AddForceElement (Args &&... args)
  7. AddJointActuator (const std::string &name, const Joint< T > &joint, double effort_limit=std::numeric_limits< double >::infinity())
  8. AddModelInstance (const std::string &name)
  9. Finalize ()
- Geometry
  1. RegisterAsSourceForSceneGraph (geometry::SceneGraph< T > *scene_graph)
  2. RegisterVisualGeometry
  3. GetVisualGeometriesForBody (const Body< T > &body) 
  4. RegisterCollisionGeometry (const Body< T > &body, const math::RigidTransform< double > &X_BG, const geometry::Shape &shape, const std::string &name, geometry::ProximityProperties properties)
  5. GetCollisionGeometriesForBody (const Body< T > &body) 
  6. ExcludeCollisionGeometriesWithCollisionFilterGroupPair (const std::pair< std::string, geometry::GeometrySet > &collision_filter_group_a, const std::pair< std::string, geometry::GeometrySet > &collision_filter_group_b)
  7. CollectRegisteredGeometries (const std::vector< const Body< T > * > &bodies)
  8. GetBodyFromFrameId (geometry::FrameId frame_id) 
  9. GetBodyFrameIdIfExists (BodyIndex body_index)
  10. GetBodyFrameIdOrThrow (BodyIndex body_index)
- Contact modeling
  1. set_contact_model (ContactModel model)
  2. set_penetration_allowance (double penetration_allowance=0.001)
  3. get_contact_penalty_method_time_scale ()
  4. set_stiction_tolerance (double v_stiction=0.001)
- State access and modification
  1. GetPositionsAndVelocities (const systems::Context< T > &context) 
  2. GetMutablePositionsAndVelocities (systems::Context< T > *context) 
  3. SetPositionsAndVelocities (systems::Context< T > *context, const VectorX< T > &q_v)
  4. GetPositions (const systems::Context< T > &context) 
  5. GetMutablePositions (systems::Context< T > *context) 
  6. SetPositions (systems::Context< T > *context, const VectorX< T > &q)
  7. GetVelocities (const systems::Context< T > &context) 
  8. GetMutableVelocities (const systems::Context< T > &context, systems::State< T > *state)
  9. SetVelocities (systems::Context< T > *context, const VectorX< T > &v)
  10. SetDefaultState (const systems::Context< T > &context, systems::State< T > *state)
  11. SetRandomState (const systems::Context< T > &context, systems::State< T > *state, RandomGenerator *generator)
  12. GetActuationFromArray (ModelInstanceIndex model_instance, const Eigen::Ref< const VectorX< T >> &u) 
  13. SetActuationInArray (ModelInstanceIndex model_instance, const Eigen::Ref< const VectorX< T >> &u_instance, EigenPtr< VectorX< T >> u)
  14. GetPositionsFromArray (ModelInstanceIndex model_instance, const Eigen::Ref< const VectorX< T >> &q) 
  15. SetPositionsInArray (ModelInstanceIndex model_instance, const Eigen::Ref< const VectorX< T >> &q_instance, EigenPtr< VectorX< T >> q)
  16. GetVelocitiesFromArray (ModelInstanceIndex model_instance, const Eigen::Ref< const VectorX< T >> &v) 
  17. SetVelocitiesInArray (ModelInstanceIndex model_instance, const Eigen::Ref< const VectorX< T >> &v_instance, EigenPtr< VectorX< T >> v)
- Parameters
- Free bodies
  1. GetFloatingBaseBodies ()
  2. GetFreeBodyPose (const systems::Context< T > &context, const Body< T > &body) const
  3. SetFreeBodyPose (systems::Context< T > *context, const Body< T > &body, const math::RigidTransform< T > &X_WB) const
  4. SetDefaultFreeBodyPose (const Body< T > &body, const math::RigidTransform< double > &X_WB)
  5. GetDefaultFreeBodyPose (const Body< T > &body) const
  6. SetFreeBodySpatialVelocity (systems::Context< T > *context, const Body< T > &body, const SpatialVelocity< T > &V_WB) const
  7. SetFreeBodyRandomPositionDistribution (const Body< T > &body, const Vector3< symbolic::Expression > &position)
  8. SetFreeBodyRandomRotationDistribution (const Body< T > &body, const Eigen::Quaternion< symbolic::Expression > &rotation)
  9. SetFreeBodyRandomRotationDistributionToUniform (const Body< T > &body)
  10. SetFreeBodyPoseInWorldFrame (systems::Context< T > *context, const Body< T > &body, const math::RigidTransform< T > &X_WB) const
  11. SetFreeBodyPoseInAnchoredFrame (systems::Context< T > *context, const Frame< T > &frame_F, const Body< T > &body, const math::RigidTransform< T > &X_FB) const
- Kinematics and dynamics
  1. EvalBodyPoseInWorld (const systems::Context< T > &context, const Body< T > &body_B) const
  2. EvalBodySpatialVelocityInWorld (const systems::Context< T > &context, const Body< T > &body_B) const
  3. ...
- System matrices
- Introspection

### Model Instances

### System dynamics

状态变量：$x = [q; v]$

系统方程：$\dot{x} = f(t, x, u)$

动力学方程：

$$
\begin{aligned}
  \dot{q} & = N(q)v\\
  M(q)\dot{v} + C(q, v)v & = \tau
\end{aligned}
$$

### Loading models from SDF files

```c++
MultibodyPlant<T> acrobot;
SceneGraph<T> scene_graph;
Parser parser(&acrobot, &scene_graph);
const std::string relative_name =
  "drake/multibody/benchmarks/acrobot/acrobot.sdf";
const std::string full_name = FindResourceOrThrow(relative_name);
parser.AddModelFromFile(full_name);
```

### Working with SceneGraph
#### Adding a MultibodyPlant connected to a SceneGraph to your Diagram
```c++
MultibodyPlant<double>& plant =
    AddMultibodyPlantSceneGraph(&builder, 0.0 /* time_step */);
plant.DoFoo(...);
```

```c++
auto items = AddMultibodyPlantSceneGraph(&builder, 0.0 /* time_step */);
items.plant.DoFoo(...);
items.scene_graph.DoBar(...);
```

```c++
auto [plant, scene_graph] = AddMultibodyPlantSceneGraph(&builder, 0.0);
...
plant.DoFoo(...);
scene_graph.DoBar(...);
```

```c++
MultibodyPlant<double>* plant{};
SceneGraph<double>* scene_graph{};
std::tie(plant, scene_graph) =
    AddMultibodyPlantSceneGraph(&builder, 0.0 /* time_step */);
plant->DoFoo(...);
scene_graph->DoBar(...);
```
#### Registering geometry with a SceneGraph
In summary, if MultibodyPlant registers collision geometry, the setup process will include:
1. Call to `RegisterAsSourceForSceneGraph()`.
2. Calls to `RegisterCollisionGeometry()`, as many as needed.
3. Call to `Finalize()`, user is done specifying the model.
4. Connect SceneGraph::get_query_output_port() to `get_geometry_query_input_port()`.

#### Accessing point contact parameters
```c++
// For a SceneGraph<T> instance called scene_graph.
const geometry::SceneGraphInspector<T>& inspector =
    scene_graph.model_inspector();
```

```c++
// For a MultibodyPlant<T> instance called mbp and a
// Context<T> called context.
const geometry::QueryObject<T>& query_object =
    mbp.get_geometry_query_input_port()
        .template Eval<geometry::QueryObject<T>>(context);
const geometry::SceneGraphInspector<T>& inspector =
    query_object.inspector();
```

```c++
// For a body with GeometryId called geometry_id
const geometry::ProximityProperties* props =
    inspector.GetProximityProperties(geometry_id);
const CoulombFriction<T>& geometry_friction =
    props->GetProperty<CoulombFriction<T>>("material",
                                           "coulomb_friction");
```

### Working with MultibodyElement parameters
```c++
MultibodyPlant<double> plant;
// ... Code to add bodies, finalize plant, and to obtain a context.
const RigidBody<double>& body =
    plant.GetRigidBodyByName("BodyName");
const SpatialInertia<double> M_BBo_B =
    body.GetSpatialInertiaInBodyFrame(context);
// .. logic to determine a new SpatialInertia parameter for body.
const SpatialInertia<double>& M_BBo_B_new = ....
// Modify the body parameter for spatial inertia.
body.SetSpatialInertiaInBodyFrame(&context, M_BBo_B_new);
```

```c++
MultibodyPlant<double> plant;
// ... Code to add bodies, finalize plant, and to obtain a
// context and a body's spatial inertia M_BBo_B.
// Scalar convert the plant.
unique_ptr<MultibodyPlant<AutoDiffXd>> plant_autodiff =
    systems::System<double>::ToAutoDiffXd(plant);
unique_ptr<Context<AutoDiffXd>> context_autodiff =
    plant_autodiff->CreateDefaultContext();
context_autodiff->SetTimeStateAndParametersFrom(context);
const RigidBody<AutoDiffXd>& body =
    plant_autodiff->GetRigidBodyByName("BodyName");
// Modify the body parameter for mass.
const AutoDiffXd mass_autodiff(mass, Vector1d(1));
body.SetMass(context_autodiff.get(), mass_autodiff);
// M_autodiff(i, j).derivatives()(0), contains the derivatives of
// M(i, j) with respect to the body's mass.
MatrixX<AutoDiffXd> M_autodiff(plant_autodiff->num_velocities(),
    plant_autodiff->num_velocities());
plant_autodiff->CalcMassMatrix(*context_autodiff, &M_autodiff);
```

### Adding modeling elements

### Modeling contact

### Energy and Power

### Finalize() stage

### References
- [Drake Concepts](https://drake.guzhaoyuan.com/introduction/drake-concept)
- [Drake Multibody](https://drake.guzhaoyuan.com/introduction/drake-multibody)
- [Drake's documentation of Multibody](https://drake.mit.edu/doxygen_cxx/classdrake_1_1multibody_1_1_multibody_plant.html#details)
- [Featherstone 2008] Featherstone, R., 2008. Rigid body dynamics algorithms. Springer.
- [Jain 2010] Jain, A., 2010. Robot and multibody dynamics: analysis and algorithms. Springer Science & Business Media.
- [Seth 2010] Seth, A., Sherman, M., Eastman, P. and Delp, S., 2010. Minimal formulation of joint motion for biomechanisms. Nonlinear dynamics, 62(1), pp.291-303.