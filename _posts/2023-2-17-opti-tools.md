---
layout: post
title: "Optimiation Tools"
categories: study
author: "Jixiang Zhang"
---

Cite: A curated list of solvers/software/frameworks relevant for dynamic optimiation.[^1]

<table border="1" class="table_legenda" width="100%">
    <tr>
        <td>Name</td>
        <td>Interface level</td>
        <td>Restrictions</td>
        <td>Algorithm</td>
        <td>Implementation language</td>
        <td>Interface languages</td>
        <td>Codegen?</td>
        <td>Dependencies</td>
        <td>License</td>
        <td>Free academic license?</td>
        <td>How to obtain license?</td>
        <td>How to install?</td>
        <td>URL</td>
        <td>Main publication</td>
        <td>Notes</td>
    </tr>
    <tr>
        <td>HPIPM</td>
        <td>QP</td>
        <td>block dense</td>
        <td>Interior-point</td>
        <td>C Assembly</td>
        <td>"Matlab, Python"</td>
        <td>No</td>
        <td>BLASFEO</td>
        <td>BSD-2</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>See readme</td>
        <td>https://github.com/giaf/hpipm</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OSQP</td>
        <td>QP</td>
        <td></td>
        <td>"ADMM, first-order, operator splitting"</td>
        <td>C</td>
        <td>"Python, Matlab, Julia, R"</td>
        <td>Yes</td>
        <td>/</td>
        <td>Apache 2</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>https://osqp.org/docs/get_started/</td>
        <td>https://osqp.org/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>FORCES</td>
        <td>"QP, OCP, MPC"</td>
        <td>"block dense, uses server (can be configured to own server)"</td>
        <td></td>
        <td></td>
        <td>"Matlab, C, C++, Python"</td>
        <td>Yes</td>
        <td>CasADi</td>
        <td></td>
        <td>yes</td>
        <td>"Online, trial license is active instantly"</td>
        <td>"Download client, add to python path"</td>
        <td>https://www.embotech.com/products/forcespro/overview/</td>
        <td></td>
        <td>Low level interface solves QCQPs!</td>
    </tr>
    <tr>
        <td>FORCESNLP</td>
        <td>MPC</td>
        <td></td>
        <td>SQP (affine inequalities online) - Interior point - ADMM</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://www.tandfonline.com/doi/full/10.1080/00207179.2017.1316017</td>
        <td>Seems to use same software-packet as FORCES</td>
    </tr>
    <tr>
        <td>HQP</td>
        <td>NLP</td>
        <td></td>
        <td>SQP</td>
        <td>C</td>
        <td>C++</td>
        <td></td>
        <td></td>
        <td>GNU GPL</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>http://omuses.github.io/hqp/doc/html/installation.html</td>
        <td>https://github.com/omuses/hqp</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>NLPQLP</td>
        <td>NLP</td>
        <td></td>
        <td>SQP</td>
        <td>Fortran</td>
        <td>"Fortran, C (by using f2c)"</td>
        <td></td>
        <td></td>
        <td></td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>http://klaus-schittkowski.de/nlpqlp.htm</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>SNOPT</td>
        <td>NLP</td>
        <td></td>
        <td>SQP</td>
        <td>Fortran 77+</td>
        <td>"C, C++, Matlab"</td>
        <td></td>
        <td></td>
        <td></td>
        <td>yes (evaluation version)</td>
        <td></td>
        <td></td>
        <td>https://ccom.ucsd.edu/~optimizers/solvers/snopt/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>IPOPT</td>
        <td>NLP</td>
        <td></td>
        <td>IP filter linesearch</td>
        <td>C++</td>
        <td>"C, C++, Fortran, Java"</td>
        <td></td>
        <td></td>
        <td>EPL</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/coin-or/Ipopt</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>WORHP</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>"C, Fortran"</td>
        <td>"C, C++, Fortran, Python, Matlab"</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://worhp.de</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>LOQO</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>C</td>
        <td>AMPL modeling language</td>
        <td></td>
        <td></td>
        <td></td>
        <td>no</td>
        <td></td>
        <td></td>
        <td>https://ampl.com/products/solvers/solvers-we-sell/loqo/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>KNITRO</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>"C, C++"</td>
        <td>"C, C++, Julia, Python, Fortran, Java, Matlab"</td>
        <td></td>
        <td></td>
        <td></td>
        <td>no</td>
        <td></td>
        <td></td>
        <td>https://www.artelys.com/solvers/knitro/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>NLOPT</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>C</td>
        <td>"C++, Fortran, Python, Matlab, OCaml, Rust, Julia, GNU R"</td>
        <td></td>
        <td></td>
        <td>GNU LGPL</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>https://nlopt.readthedocs.io/en/latest/NLopt_Installation/</td>
        <td>https://github.com/stevengj/nlopt</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Control Toolbox</td>
        <td>"OCP, MPC"</td>
        <td></td>
        <td>"iLQR, SS, MS"</td>
        <td>C++</td>
        <td>C++</td>
        <td>Yes</td>
        <td>CppAD</td>
        <td>BSD-2</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>https://github.com/ethz-adrl/control-toolbox/wiki/Quickstart</td>
        <td>https://github.com/ethz-adrl/control-toolbox</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OptimizationEngine</td>
        <td>NLP</td>
        <td></td>
        <td>First order</td>
        <td>Rust</td>
        <td>"Python, Matlab, C++"</td>
        <td></td>
        <td>CasADi</td>
        <td>Apache-2 / MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>https://alphaville.github.io/optimization-engine/docs/installation</td>
        <td>https://alphaville.github.io/optimization-engine/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>MuAO-MPC</td>
        <td>MPC</td>
        <td>linear</td>
        <td>augmented Lagrangian + nestorov</td>
        <td>"C, Python"</td>
        <td>"Python, Matlab"</td>
        <td>yes</td>
        <td></td>
        <td>BSD-3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip install muaompc</td>
        <td>http://ifatwww.et.uni-magdeburg.de/syst/muAO-MPC/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>MUSCOD II</td>
        <td>OCP</td>
        <td>Load model as FMU</td>
        <td></td>
        <td></td>
        <td>"Fortran, C, gPROMS modelling language"</td>
        <td></td>
        <td></td>
        <td></td>
        <td>Yes</td>
        <td>Contact through mail (prof Bock)</td>
        <td></td>
        <td>https://simopt.iwr.uni-heidelberg.de/muscod-ii</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ACADO</td>
        <td>MPC+OCP+NLP</td>
        <td>nonlinear</td>
        <td>"QP condensing, SQP"</td>
        <td>C++</td>
        <td>Matlab</td>
        <td>yes</td>
        <td></td>
        <td>GNU LGPL 3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>http://acado.github.io/install_linux.html</td>
        <td>http://www.acadotoolkit.org</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Mathworks MPC toolbox</td>
        <td>MPC</td>
        <td>nonlinear (since 2018b)</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>yes nonlinear (since 2020a)</td>
        <td>Fmincon+FORCES</td>
        <td></td>
        <td>no</td>
        <td></td>
        <td></td>
        <td>https://www.mathworks.com/products/model-predictive-control.html</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>YANE</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>C++</td>
        <td></td>
        <td></td>
        <td></td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>http://www.nonlinearmpc.com</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OPTCON</td>
        <td></td>
        <td></td>
        <td>multiple shooting</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://link.springer.com/article/10.1007/BF02071066</td>
        <td></td>
    </tr>
    <tr>
        <td>NMPC</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>fmincon</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>www.nmpc-book.com</td>
        <td></td>
        <td>"This is a book, referencing to YANE for code"</td>
    </tr>
    <tr>
        <td>Gekko/APMonitor</td>
        <td>OCP</td>
        <td>"nonlinear</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>modeling language"</td>
        <td></td>
        <td>"Python, Matlab, Julia (choose, not altogether)"</td>
        <td>"Python, Matlab, Julia"</td>
        <td>no (web-based interface)</td>
        <td>ipopt</td>
        <td></td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip install gekko / pip install APMonitor</td>
        <td>http://apmonitor.com/wiki/index.php/Main/PythonApp or https://github.com/BYU-PRISM/GEKKO</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>MATMPC</td>
        <td>MPC</td>
        <td>nonlinear</td>
        <td></td>
        <td>"Matlab, C"</td>
        <td>Matlab</td>
        <td>yes</td>
        <td></td>
        <td>GNU GPL 3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>"Just use the files in repo, nothing needs to be installed"</td>
        <td>https://github.com/chenyutao36/MATMPC</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>doMPC</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td></td>
        <td>HPIPM</td>
        <td>GNU LGPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/do-mpc/do-mpc</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ACADOS</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>C</td>
        <td>"C, Python, Matlab (& Octave)"</td>
        <td>yes</td>
        <td></td>
        <td>BSD-2</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/acados/acados</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OpenOCL</td>
        <td>OCP</td>
        <td>nonlinear</td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>yes</td>
        <td>CasADi+ACADOS</td>
        <td>BSD-3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td>https://openocl.org/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>YOP</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td></td>
        <td>CasADi</td>
        <td>GNU GPL 3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td>https://www.yoptimization.com/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>MPC tools</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Octave (& Matlab)</td>
        <td></td>
        <td>CasADi</td>
        <td>GNU LGPL</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://sites.engineering.ucsb.edu/~jbraw/software/mpctools/index.html</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Rockit</td>
        <td>OCP</td>
        <td>nonlinear</td>
        <td></td>
        <td>Python</td>
        <td></td>
        <td></td>
        <td>CasADi</td>
        <td>GNU LGPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://gitlab.kuleuven.be/meco-software/rockit</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Trajopt</td>
        <td>MPC</td>
        <td>robotics</td>
        <td>"SQP, box trust region, Lie algebra, signed distance using penetration depth, swept volume, multiples shooitng and direct collocation"</td>
        <td>C++</td>
        <td>"Python, C++"</td>
        <td></td>
        <td>"Gurobi, Eigen, Boost"</td>
        <td>BSD-2</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>http://rll.berkeley.edu/trajopt/doc/sphinx_build/html/</td>
        <td>http://www.eecs.berkeley.edu/~joschu/docs/trajopt-paper.pdf</td>
        <td>No updates happened in the last 6 years? CMake written for python2 ...</td>
    </tr>
    <tr>
        <td>ParNMPC</td>
        <td>MPC</td>
        <td>nonlinear</td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>Yes</td>
        <td>many Matlab toolboxes</td>
        <td>BSD-2</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/deng-haoyang/ParNMPC</td>
        <td>https://www.tandfonline.com/doi/full/10.1080/00207179.2020.1798019</td>
        <td></td>
    </tr>
    <tr>
        <td>Drake</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>"Python, C++"</td>
        <td></td>
        <td></td>
        <td>BSD-3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://drake.mit.edu</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>GPOPS</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>GPOPS-II</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>"Matlab, C++ (CGPOPS)"</td>
        <td></td>
        <td></td>
        <td></td>
        <td>no</td>
        <td>On website</td>
        <td></td>
        <td>https://www.gpops2.com</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>GRAMPC</td>
        <td>MPC</td>
        <td></td>
        <td>augmented Lagrangian</td>
        <td>C</td>
        <td>Matlab Simulink</td>
        <td></td>
        <td></td>
        <td>BSD</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://sourceforge.net/projects/grampc/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ALTRO</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td>Julia</td>
        <td>Julia</td>
        <td></td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/RoboticExplorationLab/Altro.jl</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>GusTO</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td></td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/firedrakeproject/gusto</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OBCA</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td>Julia</td>
        <td>Julia</td>
        <td></td>
        <td></td>
        <td>GNU GPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/XiaojingGeorgeZhang/OBCA</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>CIAO</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://arxiv.org/pdf/2001.05449</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>VIATOC</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>C</td>
        <td>C++</td>
        <td>yes</td>
        <td></td>
        <td>GNU LGPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://sourceforge.net/p/viatoc/wiki/Home/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Tenscalc</td>
        <td>NLP</td>
        <td></td>
        <td>Interior-point</td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>Yes</td>
        <td>"FunParTools, CmexTools, SuiteSparse"</td>
        <td></td>
        <td>yes? (see notes)</td>
        <td></td>
        <td></td>
        <td>https://github.com/hespanha/tenscalc</td>
        <td>https://link.springer.com/article/10.1007/s12532-022-00216-2</td>
        <td>"License: you are prohibited from further transferring the</td>
    </tr>
    <tr>
        <td>Software -- including any derivatives you make thereof -- to any</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>person or entity without Institution’s prior written</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>permission."</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>OPTIPLAN</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>no</td>
        <td></td>
        <td>GNU GPL 2</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/kvasnica/optiplan</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ASTOS</td>
        <td>Trajectory optimization</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://www.astos.de/products/astos</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>SOCS</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ITOMP</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>C++</td>
        <td></td>
        <td></td>
        <td>BSD-3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/Chpark/itomp</td>
        <td>http://gamma.cs.unc.edu/ITOMP/wafr12.pdf</td>
        <td></td>
    </tr>
    <tr>
        <td>Tomlab</td>
        <td>"OCP (PROPT), NLP (others)"</td>
        <td></td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td></td>
        <td></td>
        <td></td>
        <td>no</td>
        <td></td>
        <td></td>
        <td>https://tomopt.com/tomlab/about/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>MPT</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>Yes</td>
        <td></td>
        <td>GNU GPL</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://www.mpt3.org</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>RobCoGen</td>
        <td>Rigid body library with OCP extensions</td>
        <td></td>
        <td></td>
        <td>Java</td>
        <td>Kinematics-DSL</td>
        <td>Yes (C++ & Octave)</td>
        <td></td>
        <td>BSD-3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://bitbucket.org/robcogenteam/robcogen/src/master/</td>
        <td>https://doi.org/10.3929/ethz-b-000274142</td>
        <td></td>
    </tr>
    <tr>
        <td>helperOC</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>Nothing to obtain</td>
        <td></td>
        <td></td>
        <td>https://github.com/HJReachability/helperOC</td>
        <td></td>
    </tr>
    <tr>
        <td>FaSTrack</td>
        <td>motion oplanning</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>FROST</td>
        <td></td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>matlab</td>
        <td>yes</td>
        <td></td>
        <td>BSD-3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/ayonga/frost-dev</td>
        <td>https://ieeexplore.ieee.org/document/8202230</td>
        <td></td>
    </tr>
    <tr>
        <td>OptimTraj</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td></td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td></td>
        <td>https://github.com/MatthewPeterKelly/OptimTraj</td>
        <td></td>
    </tr>
    <tr>
        <td>Realtime robotics</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://rtr.ai</td>
        <td></td>
    </tr>
    <tr>
        <td>CDAL</td>
        <td>QP</td>
        <td></td>
        <td>first-order factorisation free super compact</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://arxiv.org/abs/2109.10205</td>
        <td></td>
    </tr>
    <tr>
        <td>TrajectoryOptimization.jl</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Julia</td>
        <td>Julia</td>
        <td>No</td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td>https://github.com/RoboticExplorationLab/TrajectoryOptimization.jl</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Altro.jl</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Julia</td>
        <td>Julia</td>
        <td></td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td>https://github.com/RoboticExplorationLab/Altro.jl</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ODYS QP</td>
        <td>QP</td>
        <td></td>
        <td></td>
        <td>C</td>
        <td>"C, C++, Matlab, Python"</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://www.odys.it/embedded-mpc/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Alpaqa</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>"Python, C++"</td>
        <td>Yes</td>
        <td></td>
        <td>GNU LGPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/kul-optec/alpaqa</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>EMSO</td>
        <td>modeling</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://www.enq.ufrgs.br/trac/alsoc/wiki/EMSO</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Plantegrity MPC</td>
        <td>commericial NMPC</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://www.ciengis.com/plantegrity/</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>qpSWIFT</td>
        <td>QP</td>
        <td></td>
        <td></td>
        <td>C</td>
        <td>"Python, Matlab"</td>
        <td>Yes</td>
        <td></td>
        <td>GNU GPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/qpSWIFT/qpSWIFT</td>
        <td>https://ieeexplore.ieee.org/document/8754693</td>
        <td></td>
    </tr>
    <tr>
        <td>Robotoc</td>
        <td>OCP+MPC</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>Python</td>
        <td></td>
        <td>Pinocchio</td>
        <td>BSD-3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/mayataka/robotoc</td>
        <td>https://arxiv.org/abs/2112.07232</td>
        <td></td>
    </tr>
    <tr>
        <td>Crocoddyl</td>
        <td>OCP</td>
        <td>Currently no state-only constraint possible</td>
        <td>DDP (Differential dynamic programming)</td>
        <td>C++</td>
        <td>"Python, C++"</td>
        <td>"Yes, only for C++"</td>
        <td>"pinocchio, eigen, eigenpy, boost"</td>
        <td>BSD-3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>conda install crocoddyl -c conda-forge (version is not always the newest)</td>
        <td>https://github.com/loco-3d/crocoddyl/tree/devel</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Bioptim</td>
        <td>OCP</td>
        <td></td>
        <td>Interior-point</td>
        <td>Python</td>
        <td>Python</td>
        <td></td>
        <td>CasADi+Biorbd+Rbdl</td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>conda install bioptim -c conda-forge</td>
        <td>https://github.com/pyomeca/bioptim</td>
        <td>https://ieeexplore.ieee.org/document/9808374</td>
        <td></td>
    </tr>
    <tr>
        <td>OCS2</td>
        <td>OCP</td>
        <td></td>
        <td>"SLQ DDP,iLQR DDP,HPIPM-based SQP,PISOC"</td>
        <td>C++</td>
        <td>C++</td>
        <td></td>
        <td></td>
        <td>BSD-3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td>https://github.com/leggedrobotics/ocs2</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>UNO</td>
        <td>NLP</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>"C++, AMPL modelling language (.mod file extension)"</td>
        <td></td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/cvanaret/uno</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Horizon</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td></td>
        <td></td>
        <td>"CasADi, Pinocchio, OSQP"</td>
        <td>LGPL</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/ADVRHumanoids/horizon</td>
        <td>https://www.frontiersin.org/articles/10.3389/frobt.2022.899025/full</td>
        <td></td>
    </tr>
    <tr>
        <td>MLI</td>
        <td>MPC</td>
        <td></td>
        <td>MLI (Multi level iteration schemes)</td>
        <td>MATLAB</td>
        <td>MATLAB</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://simopt.iwr.uni-heidelberg.de/mli-/-stmli</td>
        <td>http://www.ub.uni-heidelberg.de/archiv/24402</td>
        <td></td>
    </tr>
    <tr>
        <td>DYMOS</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td></td>
        <td>OpenMDAO</td>
        <td>Apache 2</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip</td>
        <td>https://github.com/OpenMDAO/dymos</td>
        <td>https://joss.theoj.org/papers/10.21105/joss.02809</td>
        <td></td>
    </tr>
    <tr>
        <td>PSOPT</td>
        <td>OCP</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>C++</td>
        <td>No</td>
        <td>"IPOPT, ADOL-C, Eigen3"</td>
        <td>GNU LGPL 2.1</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>CMake</td>
        <td>https://github.com/PSOPT/psopt</td>
        <td>https://ieeexplore.ieee.org/document/5612676</td>
        <td></td>
    </tr>
    <tr>
        <td>NOSNOC</td>
        <td>OCP</td>
        <td>Nonsmooth systems</td>
        <td>FESD (Finite elements with switch detection)</td>
        <td>MATLAB</td>
        <td>"MATLAB, (in future also Python)"</td>
        <td></td>
        <td>CasADi</td>
        <td>GNU GPL 3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>Add to MATLAB path</td>
        <td>https://github.com/nurkanovic/nosnoc</td>
        <td>https://ieeexplore.ieee.org/abstract/document/9792317</td>
        <td></td>
    </tr>
    <tr>
        <td>qpDUNES</td>
        <td>QP</td>
        <td>Block-banded convex QPs</td>
        <td>Dual newton strategy</td>
        <td>C</td>
        <td>"C, C++, MATLAB"</td>
        <td>No</td>
        <td></td>
        <td>GNU LGPL 2.1</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>Linux: apt-get. Rest: Make</td>
        <td>https://github.com/jfrasch/qpDUNES</td>
        <td>https://optimization-online.org/2013/11/4114/</td>
        <td>No updates since 2016</td>
    </tr>
    <tr>
        <td>pyMPC</td>
        <td>MPC</td>
        <td>Linear constrained MPC</td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td>No</td>
        <td>OSQP</td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip</td>
        <td>https://github.com/forgi86/pyMPC</td>
        <td>https://arxiv.org/pdf/1911.13021</td>
        <td></td>
    </tr>
    <tr>
        <td>HILO-MPC</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td>No</td>
        <td></td>
        <td>GNU LGPL 2</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip</td>
        <td>https://github.com/hilo-mpc/hilo-mpc</td>
        <td>https://arxiv.org/abs/2203.13671</td>
        <td>"Also does estimation, machine learning & modelling"</td>
    </tr>
    <tr>
        <td>OMPL</td>
        <td>Motion planning</td>
        <td>Sampling-based</td>
        <td></td>
        <td>C++</td>
        <td>"C++, Python"</td>
        <td>Yes</td>
        <td>"Boost, CMake, Eigen"</td>
        <td>BSD-3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>CMake</td>
        <td>https://github.com/ompl/ompl</td>
        <td>https://ompl.kavrakilab.org/core/ieee-ram-2012-ompl.pdf</td>
        <td></td>
    </tr>
    <tr>
        <td>CVXGEN</td>
        <td>QP</td>
        <td>Must be convex</td>
        <td></td>
        <td>C</td>
        <td>"C, Matlab"</td>
        <td>Yes</td>
        <td></td>
        <td>Custom license</td>
        <td>yes</td>
        <td>Request through form (https://cvxgen.com/licenses/new)</td>
        <td>CMake (Matlab)/Make (C)</td>
        <td>https://cvxgen.com/docs/index.html</td>
        <td>https://stanford.edu/~boyd/papers/pdf/code_gen_impl.pdf</td>
        <td>No updates since 2014</td>
    </tr>
    <tr>
        <td>YALMIP</td>
        <td>QP</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td></td>
        <td></td>
        <td>Custom license</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>Add to MATLAB path</td>
        <td>https://yalmip.github.io/tutorial/basics/</td>
        <td>https://ieeexplore.ieee.org/document/1393890</td>
        <td>Automatically selects solver</td>
    </tr>
    <tr>
        <td>libmpc++</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>C++</td>
        <td>C++</td>
        <td>No</td>
        <td>"Eigen3, NLopt, OSQP"</td>
        <td>"Not specified, open source"</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td></td>
        <td>https://github.com/nicolapiccinelli/libmpc</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>omsf</td>
        <td>Motion planning</td>
        <td>Only discrete dynamics supported</td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td>No</td>
        <td>ipopt</td>
        <td>GNU LGPL 3</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip</td>
        <td>https://github.com/mkatliar/omsf</td>
        <td>https://www.sciencedirect.com/science/article/abs/pii/S1369847818308544</td>
        <td></td>
    </tr>
    <tr>
        <td>PythonLinearNonlinearControl</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td>No</td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip</td>
        <td>https://github.com/Shunichi09/PythonLinearNonlinearControl</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Robust tube MPC</td>
        <td>MPC</td>
        <td>Inequality constraints are convex</td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>No</td>
        <td>"Optimization toolbox, control toolbox, MPT3"</td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>Add to MATLAB path</td>
        <td>https://github.com/HiroIshida/robust-tube-mpc</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>Autogenu-jupyter</td>
        <td>MPC</td>
        <td></td>
        <td>C/GMRES</td>
        <td>"C++, Python"</td>
        <td>"Python, C++"</td>
        <td>Yes</td>
        <td></td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip & CMake</td>
        <td>https://github.com/mayataka/autogenu-jupyter</td>
        <td>http://ifatwww.et.uni-magdeburg.de/ifac2020/media/pdfs/0236.pdf</td>
        <td></td>
    </tr>
    <tr>
        <td>NASOQ</td>
        <td>QP</td>
        <td>Mainly focused on sparse problems</td>
        <td></td>
        <td>C++</td>
        <td>"C++, MATLAB"</td>
        <td>No</td>
        <td>optional: BLAS & LAPACK</td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>CMake</td>
        <td>https://github.com/sympiler/nasoq</td>
        <td>https://nasoq.github.io/siggraph20.html</td>
        <td></td>
    </tr>
    <tr>
        <td>Trajopt-Hanyas</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>"C++, Python"</td>
        <td>"C++, Python"</td>
        <td>No</td>
        <td>"Pybind, OpenBLAS, Armadillo"</td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>pip</td>
        <td>https://github.com/hanyas/trajopt</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>NMPC-DCLF-DCBF</td>
        <td>MPC</td>
        <td></td>
        <td></td>
        <td>Matlab</td>
        <td>Matlab</td>
        <td>No</td>
        <td>"Yalmip, ipopt"</td>
        <td>MIT</td>
        <td>yes</td>
        <td>Nothing to obtain</td>
        <td>Add to MATLAB path</td>
        <td>https://github.com/HybridRobotics/NMPC-DCLF-DCBF</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>SLEQP</td>
        <td>NLP</td>
        <td>Sparse</td>
        <td></td>
        <td>C</td>
        <td>C</td>
        <td></td>
        <td>(Gurobi|HIGHS|SoPplex) (Cholmod|UMFPACK|MUMPS|MA*|SPQR) trlib</td>
        <td>LGPL 3</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/chrhansk/sleqp</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>DAQP</td>
        <td>QP</td>
        <td>Dense</td>
        <td>dual active set</td>
        <td>C</td>
        <td>C</td>
        <td></td>
        <td>none</td>
        <td>MIT</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/darnstrom/daqp</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>ProxSuite</td>
        <td>QP</td>
        <td>Dense or sparse</td>
        <td>primal-dual proximal</td>
        <td>C++</td>
        <td>C</td>
        <td></td>
        <td>Eigen</td>
        <td>BSD-2</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/Simple-Robotics/proxsuite</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>SymForce</td>
        <td></td>
        <td></td>
        <td></td>
        <td>Python</td>
        <td>Python</td>
        <td>yes</td>
        <td>"SymEngine, SymPy, Eigen, NumPy"</td>
        <td>Apache-2</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>https://github.com/symforce-org/symforce</td>
        <td>https://doi.org/10.15607/RSS.2022.XVIII.041</td>
        <td></td>
    </tr>
    <tr>
        <td>CHOMP</td>
        <td>Motion planning</td>
        <td>Unconstrained</td>
        <td>"unconstrained Newton step with Hess part coming from L2 distance wrt to previous iteration, distance transform, joint limits in objective or projection in-bounds, obstacle codes is penalty integrated over path length"</td>
        <td>C++</td>
        <td></td>
        <td>no</td>
        <td></td>
        <td>BSD</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td>http://www.nathanratliff.com/thesis-research/chomp</td>
        <td>http://dx.doi.org/10.1109/ROBOT.2009.5152817</td>
        <td>incorporated in MoveIt</td>
    </tr>
    <tr>
        <td>STOMP</td>
        <td>Motion planning</td>
        <td>Unconstrained</td>
        <td>"derivative free optimization, obstacles: self= collection of spheres"</td>
        <td>C++</td>
        <td></td>
        <td>no</td>
        <td></td>
        <td>BSD</td>
        <td>yes</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>SBPL</td>
        <td>Motion planning</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>GPMP</td>
        <td>Motion planning</td>
        <td>Unconstrained</td>
        <td>"State traj is Gaussian Process (states may be sampled at different freq), smells like Matrix-Lyap diff eq, Mahalanobis distance, integral of collision cost, unconstrained Newton step with Hess part coming from L2 distance wrt to previous iteration, emphasis on fast interpolation with GP, constraints handled by r projection in-bounds"</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>GPMP2</td>
        <td></td>
        <td></td>
        <td>Nonlinear least squares</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>BSD</td>
        <td></td>
        <td></td>
        <td></td>
        <td>https://github.com/gtrll/gpmp2</td>
        <td>http://www.cc.gatech.edu/~bboots3/files/GPMP2.pdf</td>
        <td></td>
    </tr>
</table>

[^1]: https://github.com/ADVRHumanoids/robot_inertia_publisher
