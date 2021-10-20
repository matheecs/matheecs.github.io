---
layout: post
title: "ODE 四足机器人仿真"
categories: study
author: "Jixiang Zhang"
---

![](/images/ode_leg.gif)

```c++
// �ȒP�I���H�I���{�b�g�V�~�����[�V����
// Open Dynamics Engine�ɂ�郍�{�b�g�v���O���~���O
// �o��������, �X�k�o�� (2007) http://demura.net/
// ���̃v���O�����͏�{�̃T���v���v���O�����ł��D
// �v���O���� 8.1:  4�r���{�b�g���s�v���O����
// This program is a sample program of my book as follows
//�gRobot Simulation - Robot programming with Open Dynamics Engine,
// (260pages, ISBN:978-4627846913, Morikita Publishing Co. Ltd.,
// Tokyo, 2007)�h by Kosei Demura, which is written in Japanese (sorry).
// http://demura.net/simulation
// Please use this program if you like. However it is no warranty.
// legged.cpp by Kosei Demura (2007-2008)
// �X�V���� (Change log)
// 2008-10-3: dWorldSetCFM(world, 1e-3),dWorldSetERP(world, 0.8)�̒ǉ� (add)
// 2008-7-7: dInitODE(),dCloseODE()�̒ǉ� (add)
#include <drawstuff/drawstuff.h>
#include <ode/ode.h>
#include <stdio.h>
#include <stdlib.h>

#ifdef _MSC_VER
#pragma warning(disable : 4244 4305)  // for VC++, no precision loss complaints
#endif

#ifdef dDOUBLE
#define dsDrawCapsule dsDrawCapsuleD
#define dsDrawBox dsDrawBoxD
#define dsDrawLine dsDrawLineD
#endif

dWorldID world;  // ���͊w�v�Z�p�̃��[���h (for dynamics)
dSpaceID space;  // �Փˌ��o�p�̃X�y�[�X (for collision)
dGeomID ground;  // �n�� (ground)
dJointGroupID contactgroup;  // �ڐG�_�O���[�v (contact group for collision)
dsFunctions fn;  // �h���[�X�^�b�t�̕`��֐� (function of
                 // drawstuff)

#define LINK_NUM 3  // �S�����N�� (total number of links)
#define JT_NUM 3  // �S�W���C���g�� (total number of joints)
#define LEG_NUM 4  // �S�r�� (total number of legs)

typedef struct {
  dBodyID body;
  dGeomID geom;
  dJointID joint;
  dReal m, r, x, y, z;  // ����(weight)�C���a(radius)�C�ʒu(positin:x,y,z)
} MyLink;

MyLink leg[LEG_NUM][LINK_NUM], torso;  // �r(leg)�C����(torso)

dReal THETA[LEG_NUM][LINK_NUM] = {{0}, {0}, {0}, {0}};  // �ڕW�p�x(target angle)
dReal SX = 0, SY = 0, SZ = 0.65;  // ���̏d�S�̏����ʒu(initial positon of COG)
dReal gait[12][LEG_NUM][JT_NUM];  // �ڕW�p�x (target angle of gait)
dReal l1 = 1.5 * 0.05, l2 = 0.3, l3 = 0.3;  // �����N�� 0.25 (lenth of links)

dReal lx = 0.5, ly = 0.3, lz = 0.05;             // body sides
dReal r1 = 0.02, r2 = 0.02, r3 = 0.02;           // leg radius
dReal cx1 = (lx - r1) / 2, cy1 = (ly + l1) / 2;  // �ꎞ�ϐ� (temporal variable)
dReal c_x[LEG_NUM][LINK_NUM] = {
    {cx1, cx1, cx1},
    {-cx1, -cx1, -cx1},  // �֐ߒ��Sx���W(center of joints)
    {-cx1, -cx1, -cx1},
    {cx1, cx1, cx1}};
dReal c_y[LEG_NUM][LINK_NUM] = {
    {cy1, cy1, cy1},
    {cy1, cy1, cy1},  // �֐ߒ��Sy���W(center of joints)
    {-cy1, -cy1, -cy1},
    {-cy1, -cy1, -cy1}};
dReal c_z[LEG_NUM][LINK_NUM] = {
    {0, 0, -l2},  // �֐ߒ��Sz���W(center of joints)
    {0, 0, -l2},
    {0, 0, -l2},
    {0, 0, -l2}};

/*** ���{�b�g�̐��� ***/
void makeRobot() {
  dReal torso_m = 10.0;  // ���̂̎��� (weight of torso)
  dReal l1m = 0.005, l2m = 0.5, l3m = 0.5;  // �����N�̎��� (weight of links)

  dReal x[LEG_NUM][LINK_NUM] = {
      {cx1, cx1, cx1},
      {-cx1, -cx1, -cx1},  // �e�����N�̈ʒu(link position, x���W)
      {-cx1, -cx1, -cx1},
      {cx1, cx1, cx1}};
  dReal y[LEG_NUM][LINK_NUM] = {
      {cy1, cy1, cy1},
      {cy1, cy1, cy1},  // �e�����N�̈ʒu(link position, y���W)
      {-cy1, -cy1, -cy1},
      {-cy1, -cy1, -cy1}};
  dReal z[LEG_NUM][LINK_NUM] = {
      // �e�����N�̈ʒu(link position, z���W)
      {c_z[0][0], (c_z[0][0] + c_z[0][2]) / 2, c_z[0][2] - l3 / 2},
      {c_z[0][0], (c_z[0][0] + c_z[0][2]) / 2, c_z[0][2] - l3 / 2},
      {c_z[0][0], (c_z[0][0] + c_z[0][2]) / 2, c_z[0][2] - l3 / 2},
      {c_z[0][0], (c_z[0][0] + c_z[0][2]) / 2, c_z[0][2] - l3 / 2}};
  dReal r[LINK_NUM] = {r1, r2, r3};  // �����N�̔��a (radius of links)
  dReal length[LINK_NUM] = {l1, l2, l3};  // �����N�̒��� (length of links)
  dReal weight[LINK_NUM] = {l1m, l2m, l3m};  // �����N�̎��� (weight of links)
  dReal axis_x[LEG_NUM][LINK_NUM] = {
      {0, 1, 0}, {0, 1, 0}, {0, 1, 0}, {0, 1, 0}};
  dReal axis_y[LEG_NUM][LINK_NUM] = {
      {1, 0, 1}, {1, 0, 1}, {1, 0, 1}, {1, 0, 1}};
  dReal axis_z[LEG_NUM][LINK_NUM] = {
      {0, 0, 0}, {0, 0, 0}, {0, 0, 0}, {0, 0, 0}};

  // ���̂̐��� (crate a torso)
  dMass mass;
  torso.body = dBodyCreate(world);
  dMassSetZero(&mass);
  dMassSetBoxTotal(&mass, torso_m, lx, ly, lz);
  dBodySetMass(torso.body, &mass);
  torso.geom = dCreateBox(space, lx, ly, lz);
  dGeomSetBody(torso.geom, torso.body);
  dBodySetPosition(torso.body, SX, SY, SZ);

  // �r�̐��� (create 4 legs)
  dMatrix3 R;                                // ��]�s��
  dRFromAxisAndAngle(R, 1, 0, 0, M_PI / 2);  // 90�x��]���n�ʂƕ��s
  for (int i = 0; i < LEG_NUM; i++) {
    for (int j = 0; j < LINK_NUM; j++) {
      leg[i][j].body = dBodyCreate(world);
      if (j == 0) dBodySetRotation(leg[i][j].body, R);
      dBodySetPosition(leg[i][j].body, SX + x[i][j], SY + y[i][j],
                       SZ + z[i][j]);
      dMassSetZero(&mass);
      dMassSetCapsuleTotal(&mass, weight[j], 3, r[j], length[j]);
      dBodySetMass(leg[i][j].body, &mass);
      leg[i][j].geom = dCreateCapsule(space, r[j], length[j]);
      dGeomSetBody(leg[i][j].geom, leg[i][j].body);
    }
  }

  // �W���C���g�̐����ƃ����N�ւ̎��t�� (create links and attach legs to the torso)
  for (int i = 0; i < LEG_NUM; i++) {
    for (int j = 0; j < LINK_NUM; j++) {
      leg[i][j].joint = dJointCreateHinge(world, 0);
      if (j == 0)
        dJointAttach(leg[i][j].joint, torso.body, leg[i][j].body);
      else
        dJointAttach(leg[i][j].joint, leg[i][j - 1].body, leg[i][j].body);
      dJointSetHingeAnchor(leg[i][j].joint, SX + c_x[i][j], SY + c_y[i][j],
                           SZ + c_z[i][j]);
      dJointSetHingeAxis(leg[i][j].joint, axis_x[i][j], axis_y[i][j],
                         axis_z[i][j]);
    }
  }
}

/*** ���{�b�g�̕`�� (draw a robot) ***/
void drawRobot() {
  dReal r, length;
  dVector3 sides;

  // ���̂̕`��
  dsSetColor(1.3, 1.3, 1.3);
  dGeomBoxGetLengths(torso.geom, sides);
  dsDrawBox(dBodyGetPosition(torso.body), dBodyGetRotation(torso.body), sides);

  // �r�̕`��
  for (int i = 0; i < LEG_NUM; i++) {
    for (int j = 0; j < LINK_NUM; j++) {
      dGeomCapsuleGetParams(leg[i][j].geom, &r, &length);
      if (j == 0)
        dsDrawCapsule(dBodyGetPosition(leg[i][j].body),
                      dBodyGetRotation(leg[i][j].body), 0.5 * length, 1.2 * r);
      else
        dsDrawCapsule(dBodyGetPosition(leg[i][j].body),
                      dBodyGetRotation(leg[i][j].body), length, r);
    }
  }
}

static void nearCallback(void *data, dGeomID o1, dGeomID o2) {
  dBodyID b1 = dGeomGetBody(o1), b2 = dGeomGetBody(o2);
  if (b1 && b2 && dAreConnectedExcluding(b1, b2, dJointTypeContact)) return;
  // if ((o1 != ground) && (o2 != ground)) return;

  static const int N = 20;
  dContact contact[N];
  int n = dCollide(o1, o2, N, &contact[0].geom, sizeof(dContact));
  if (n > 0) {
    for (int i = 0; i < n; i++) {
      contact[i].surface.mode = dContactSoftERP | dContactSoftCFM;
      contact[i].surface.mu = 0.8;  // 2.0;
      contact[i].surface.soft_erp = 0.9;
      contact[i].surface.soft_cfm = 1e-5;
      dJointID c = dJointCreateContact(world, contactgroup, &contact[i]);
      dJointAttach(c, b1, b2);
    }
  }
}

/*** �t�^���w�̌v�Z (calculate inverse kinematics ***/
void inverseKinematics(dReal x, dReal y, dReal z, dReal *ang1, dReal *ang2,
                       dReal *ang3, int posture) {
  dReal l1a = 0, l3a = l3 + r3 / 2;

  double c3 = (x * x + z * z + (y - l1a) * (y - l1a) - (l2 * l2 + l3a * l3a)) /
              (2 * l2 * l3a);
  double s2 = (y - l1a) / (l2 + l3a * c3);
  double c2 = sqrt(1 - s2 * s2);
  double c1 = (l2 + l3a * c3) * c2 / sqrt(x * x + z * z);
  // printf("c3=%f s2=%f c2=%f c1=%f \n", c3,s2,c2,c1);
  if (sqrt(x * x + y * y + z * z) > l2 + l3) {
    printf(" Target point is out of range \n");
  }

  switch (posture) {
    case 1:  // �p���P (posture 1)
      *ang1 = atan2(x, -z) - atan2(sqrt(1 - c1 * c1), c1);
      *ang2 = -atan2(s2, c2);
      *ang3 = atan2(sqrt(1 - c3 * c3), c3);
      break;
    case 2:  // �p���Q (posture 2)
      *ang1 = atan2(x, -z) + atan2(sqrt(1 - c1 * c1), c1);
      *ang2 = -atan2(s2, c2);
      *ang3 = -atan2(sqrt(1 - c3 * c3), c3);
      break;
    case 3:  // �p���R (posture 3)
      *ang1 = M_PI + (atan2(x, -z) - atan2(sqrt(1 - c1 * c1), c1));
      *ang2 = -M_PI + atan2(s2, c2);
      *ang3 = -atan2(sqrt(1 - c3 * c3), c3);
      break;
    case 4:  // �p���S (posture 4)
      *ang1 = M_PI + atan2(x, -z) + atan2(sqrt(1 - c1 * c1), c1);
      *ang2 = -M_PI + atan2(s2, c2);
      *ang3 = atan2(sqrt(1 - c3 * c3), c3);
      break;
  }
}

/*** P���� (P control) ***/
void Pcontrol() {
  dReal kp = 2.0, fMax = 100.0;

  for (int i = 0; i < LEG_NUM; i++) {
    for (int j = 0; j < LINK_NUM; j++) {
      dReal tmp = dJointGetHingeAngle(leg[i][j].joint);
      dReal diff = THETA[i][j] - tmp;
      dReal u = kp * diff;
      dJointSetHingeParam(leg[i][j].joint, dParamVel, u);
      dJointSetHingeParam(leg[i][j].joint, dParamFMax, fMax);
    }
  }
}

/*** ���s���� (gait control) ***/
void walk() {
  static int t = 0, steps = 0;
  int interval = 50;

  if ((steps++ % interval) == 0) {
    t++;
  } else {  // �ڕW�֐ߊp�x�̐ݒ� (set target gait angles)
    for (int leg_no = 0; leg_no < LEG_NUM; leg_no++) {
      for (int joint_no = 0; joint_no < JT_NUM; joint_no++) {
        THETA[leg_no][joint_no] = gait[t % 12][leg_no][joint_no];
      }
    }
  }
  Pcontrol();  // P���� (P control)
}

/*** �V�~�����[�V�������[�v (Simulation Loop) ***/
void simLoop(int pause) {
  if (!pause) {
    walk();  // ���s���� (gait control)
    dSpaceCollide(space, 0, &nearCallback);  // �Փˌ��o (collision detection)
    dWorldStep(world, 0.01);  // �X�e�b�v�X�V (step a simulation)
    dJointGroupEmpty(contactgroup);  // �ڐG�_�O���[�v���� (empty jointgroup)
  }
  drawRobot();  // ���{�b�g�̕`�� (draw a robot)
}

/*** ���_�Ǝ����̐ݒ� (Set view point and direction) ***/
void start() {
  float xyz[3] = {1.0f, -1.2f, 0.5f};     // ���_[m] (View point)
  float hpr[3] = {121.0f, -10.0f, 0.0f};  // ����[��] (View direction)
  dsSetViewpoint(xyz, hpr);  // ���_�Ǝ����̐ݒ� (Set View point and direction)
  dsSetSphereQuality(3);
  dsSetCapsuleQuality(6);
}

/*** �h���[�X�^�b�t�̐ݒ� (Set drawstuff) ***/
void setDrawStuff() {
  fn.version = DS_VERSION;
  fn.start = &start;
  fn.step = &simLoop;
  fn.command = NULL;
  fn.path_to_textures =
      "/Users/zhangjixiang/Code/ode-0.16.2/drawstuff/textures";
}

void calcAngle() /*** �ڕW�p�x�̌v�Z (Calculate target angles) ***/
{
  dReal z0 = -0.4, z1 = -0.37;  // z0:�n�ʂ܂ł̍���(height to the
                                // ground)�Cz1:�ō����B�_ (highest point)
  dReal y1 = 0.05, fs = 0.2;  // y1:���E�̕ψ�(defference between right and
                              // left)�Cfs:���� (foot step)
  dReal f1 = fs / 4, f2 = fs / 2, f3 = 3 * fs / 4,
        f4 = fs;  // �ꎞ�ϐ� (temporal variables)
  dReal traj[12][LEG_NUM][3] = {
      // �ڕW�O���_ (trajectory points)
      // leg0: left fore leg,  leg1: left rear leg
      // leg2: right rear leg, leg3: right fore leg
      // ���Oleg0�V�r ����leg1    �E�� leg2    �E�Oleg3
      {{0, y1, z0},
       {0, y1, z0},
       {0, y1, z0},
       {0, y1, z0}},  // 0 �d�S�ړ�(move COG)
      {{f2, y1, z1}, {0, y1, z0}, {0, y1, z0}, {0, y1, z0}},  // 1 ���n(takeoff)
      {{f4, y1, z0},
       {0, y1, z0},
       {0, y1, z0},
       {0, y1, z0}},  // 2 ���n(touchdown)
      {{f3, -y1, z0},
       {-f1, -y1, z0},
       {-f1, -y1, z0},
       {-f1, -y1, z0}},  // 3 �d�S�ړ�(move COG)
      //    leg0        leg1     leg2 �V�r(swing)  leg3
      {{f3, -y1, z0},
       {-f1, -y1, z0},
       {f1, -y1, z1},
       {-f1, -y1, z0}},  // 4 ���n(takeoff)
      {{f3, -y1, z0},
       {-f1, -y1, z0},
       {f3, -y1, z0},
       {-f1, -y1, z0}},  // 5 ���n(touchdown)
      {{f2, -y1, z0},
       {-f2, -y1, z0},
       {f2, -y1, z0},
       {-f2, -y1, z0}},  // 6 �d�S�ړ�(move COG)
      //    leg0        leg1      leg2       leg3 �V�r(swing)
      {{f2, -y1, z0},
       {-f2, -y1, z0},
       {f2, -y1, z0},
       {0, -y1, z1}},  // 7 ���n(takeoff)
      {{f2, -y1, z0},
       {-f2, -y1, z0},
       {f2, -y1, z0},
       {f2, -y1, z0}},  // 8 ���n(touchdown)
      {{f1, y1, z0},
       {-f3, y1, z0},
       {f1, y1, z0},
       {f1, y1, z0}},  // 9 �d�S�ړ�(move COG)
      //   leg0    leg1 �V�r(swing)  leg2         leg3
      {{f1, y1, z0},
       {-f1, y1, z1},
       {f1, y1, z0},
       {f1, y1, z0}},  // 10 ���n(takeoff)
      {{f1, y1, z0},
       {f1, y1, z0},
       {f1, y1, z0},
       {f1, y1, z0}}  // 11 ���n(touchdown)
  };

  dReal angle1, angle2, angle3;
  int posture = 1;  // �p��(posture)
  for (int i = 0; i < 12; i++) {
    for (int k = 0; k < LEG_NUM; k++) {
      inverseKinematics(traj[i][k][0],
                        traj[i][k][1],  // �t�^���w(inverse kinematics)
                        traj[i][k][2], &angle1, &angle2, &angle3, posture);
      gait[i][k][0] = angle1;
      gait[i][k][1] = angle2;
      gait[i][k][2] = angle3;
    }
  }
}

int main(int argc, char *argv[]) {
  dInitODE();
  setDrawStuff();
  world = dWorldCreate();
  space = dHashSpaceCreate(0);
  contactgroup = dJointGroupCreate(0);
  ground = dCreatePlane(space, 0, 0, 1, 0);
  dWorldSetGravity(world, 0, 0, -9.8);
  dWorldSetCFM(world, 1e-3);  // CFM�̐ݒ� (global CFM)
  dWorldSetERP(world, 0.9);   // ERP�̐ݒ� (global ERP)
  makeRobot();
  calcAngle();
  dsSimulationLoop(argc, argv, 800, 480, &fn);
  dSpaceDestroy(space);
  dWorldDestroy(world);
  dCloseODE();
  return 0;
}
```