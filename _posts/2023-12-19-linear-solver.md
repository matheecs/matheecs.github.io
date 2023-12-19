---
layout: post
title: "Linear Solver Notes"
categories: study
author: "Jixiang Zhang"
---

Result(speed) in TrajOpti(CasADi + IPopt):

$$MA57 ≈ MUMPS >> MA86 > MA97 (default\,solver\,of\,WORHP)$$

<table>
   <tr>
      <th>Solver</th>
      <th>Free to all</th>
      <th>Free to academics</th>
      <th>Problem size</th>
      <th>Parallel</th>
      <th>Repeatable answers</th>
      <th class="wide">Notes</th>
   </tr>
   <tr>
      <td>MA27</td>
      <td><a href="https://licences.stfc.ac.uk/product/coin-hsl-archive">Yes</a></td>
      <td>Yes</td>
      <td>Small</td>
      <td>No</td>
      <td>Yes</td>
      <td class="note">Outdated, relatively slow</td>
   </tr>
   <tr>
      <td>MA57</td>
      <td></td>
      <td>Yes</td>
      <td>Small / Medium</td>
      <td>Threaded BLAS</td>
      <td>Yes</td>
   </tr>
   <tr>
      <td>HSL_MA77</td>
      <td></td>
      <td>Yes</td>
      <td>Huge</td>
      <td>Limited</td>
      <td>Yes</td>
      <td class="note">Out-of-core</td>
   </tr>
   <tr>
      <td>HSL_MA86</td>
      <td></td>
      <td>Yes</td>
      <td>Large</td>
      <td>Highly</td>
      <td>No*</td>
      <td class="note">Designed for multicore</td>
   </tr>
   <tr>
      <td>HSL_MA97</td>
      <td></td>
      <td>Yes</td>
      <td>Small / Medium / Large</td>
      <td>Yes</td>
      <td>Yes</td>
      <td class="note">Slower than HSL_MA86 on large problems</td>
   </tr>
</table>

* Note: HSL_MA86 achieves repeatable answers in serial, however when running in parallel operations may be reordered for better performance. This leads to different (but equally valid) solutions.[^1]
* For many problems scaling is not necessary.[^2]

[^1]: Coin-HSL <https://licences.stfc.ac.uk/product/coin-hsl>
[^2]: On the effects of scaling on the performance of Ipopt <https://epubs.stfc.ac.uk/manifestation/8693/RAL-P-2012-009.pdf>
