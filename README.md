## Analytical Integration of Spherical Harmonics ##

This code implements the analytical integration method from *A Closed-Form Method for Integrating Arbitrary Spherical Polynomials*.

### Use this code ###

The integration code is header only and template based. It requires to compile with a C++11 compatible compiler and uses [Eigen](http://eigen.tuxfamily.org) to perform the required linear algebra operations. Eigen is defined as a submodule of this git repository.

A CMake script is provided to perform the sanity check of the code. Examples of use of the spherical integration code can be found in `test`.

#### Templates ####

This code relies on templates to enable a simpler integration into over codebases. We provide examples of our wrappers in `tests/Test.hpp`. To use our code, you will have to define the following class wrappers:

   + `class SH` that provides `FastBasis`, a method to compute the vector of Spherical Harmonics basis elements for a given input vector and over SH related functions. See our implementation in `tests/SphericalInt.cpp`.
   + `class Vector` that represent 3D vectors. This class needs to provide static functions such as `Dot`, `Normalize` and `Cross`.
   + `class Triangle` and `class Edge` that represent a spherical triangle which is a simple interator over a set of edges.

#### Using Spherical Integration Code ####

The spherical integration code is located in the `SphericalIntegration.hpp` header. Given an input triangle `triangle` and an SH decomposition of some function to integrate `clm`, the method works as follows:

First, convert the SH expansion to a Zonal Harmonics expansion `zlm` with a set of basis vectors `basis`. You can use our `SamplingBlueNoise` method located in `DirectionsSampling.hpp` to generate a basis vectors:

      // Generate a basis vector for an order `order` SH
      const auto basis = SamplingBlueNoise<Vector>(2*order+1);


Then use this basis vectors to compute the conversion SH -> ZH -> Power of cosines:

      // Get the Zonal weights matrix and the Zlm -> Ylm conversion matrix
      // and compute the product of the two: `Prod = A x Zw`.
      const auto ZW  = ZonalWeights<Vector>(basis);
      const auto Y   = ZonalExpansion<SH, Vector>(basis);
      const auto A   = computeInverse(Y);
      const auto cpw = clm.transpose() * (A*ZW);

Note that the matrix `A*ZW` can be precomputed to improve performances.

Then the `AxialMoments` method will return the vector of cosine power integrals for the triangle `triangle`:

      // Analytical evaluation of the integral of power of cosines for
      // the different basis elements up to the order defined by the
      // number of elements in the basis
      const auto moments = AxialMoments<Triangle, Vector>(triangle, basis);
      return cpw.dot(moments);