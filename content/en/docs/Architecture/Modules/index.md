---
title: "Modules"
linkTitle: "Modules"
weight: 3
date: 2017-01-05
description: >
  A module represents an independent functional module or a combination of functional modules. Typical modules include the **topology** modules, **compute** modules, **force** modules and **constrain** modules, etc.
---

## Structure of a module

The following code demonstrates the typical strcuture of a module, which contains regular variables, input variables and output variables.

```go
template<typename TDataType>
class SummationDensity : public ComputeModule
{
  DECLARE_CLASS_1(SummationDensity, TDataType)

public:
  typedef typename TDataType::Real Real;
  typedef typename TDataType::Coord Coord;

  SummationDensity();
  ~SummationDensity() override {};

  void compute() override;

protected:
  void calculateScalingFactor();

  void compute(DeviceArray<Real>& rho);

  void compute(
    DeviceArray<Real>& rho,
    DeviceArray<Coord>& pos,
    NeighborList<int>& neighbors,
    Real smoothingLength,
    Real mass);

public:
  ///Define intrinsic fields
  DEF_EMPTY_VAR(RestDensity, Real, "Rest Density");
  DEF_EMPTY_VAR(Mass, Real, "Particle mass, note that this module only support a constant mass for all particles.");
  DEF_EMPTY_VAR(SmoothingLength, Real, "Indicating the smoothing length");
  DEF_EMPTY_VAR(SamplingDistance, Real, "Indicating the initial sampling distance");

  ///Define input fields
  /**
    * @brief Particle positions
    */
  DEF_EMPTY_IN_ARRAY(Position, Coord, DeviceType::GPU, "Particle position");

  /**
    * @brief Neighboring particles
    *
    */
  DEF_EMPTY_IN_NEIGHBOR_LIST(NeighborIndex, int, "Neighboring particles' ids");

  ///Define output fields
  /**
    * @brief Particle densities
    */
  DEF_EMPTY_OUT_ARRAY(Density, Real, DeviceType::GPU, "Particle position");

private:
  ///Define other class members
};
```

In the Qt-based GUI, the above module appears like

![ModuleView](./modules/summation-density.jpg)

Besides, all regular variables will be shown in the property editor

![Property](./modules/property.jpg)

### Intrinsic fields

> All intrinsic field names are started with an prefix **var**, e.g., this->varRestDensity().

If a intrinsic field is defined, it can be used by the module without explicitly initializing its value.

To declare an empty field of real type, use the following formulation
```
DEF_EMPTY_VAR(RestDensity, Real, "Rest Density");
```

### Input fields

> All input field names are started with an prefix **in**, e.g., this->inPosition().


Input fields of a module are usually empty. Therefore, to execuate a module, its input fields should be explicitly specified.

To set up an input field, the most efficienty way is by connecting an output field to an input field, e.g., 
```go
this->outPosition().connect(m_densitySum->inPosition());
```

### Output fields

> All output field names are started with an prefix **out**, e.g., this->outPosition().


The values of output fields are computed from input fields.
As long as either the intrinsic fields all input fields are changed, the output fields should be updated again.
