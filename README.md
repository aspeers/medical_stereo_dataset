# Medical Stereo Dataset (v 1.0)

The dataset consists of calibrated stereo image sequences of a silicone liver phantom attached to a motion control platform (one linear and one rotational stage) for precise motion control.  The phantom is rigidly affixed to a fiducial board containing 2D AR-tag fiducials as well as 3D Lego fiducials in the corners of the fiducial board.  These fiducials are used to provide an initial alignment of the reconstructed point cloud to the corresponding CT scan and for verification of the provided motion commands.  The CT scan is also included as part of the dataset.  Note: All fiducials should be masked out and not processed by a vision algorithm using this data for verification / evaluation of your system.

This dataset can be used to help evaluate surgical augmented reality / image guided surgery systems allowing for the reporting of both surface registration error and motion estimation errors.

# Dataset organization

The sequences contained within the Medical Stereo Dataset test three conditions: translation only, rotation only and translation and rotation combined.  Each test sequence consists of 31 frames evenly spaced with one degree / one mm motion increments.  The image sequences are located in the **images** subfolder of the
dataset.  Calibration information is found in plain-text format within the **calibration** subfolder.  Naming conventions of the matrices follow Open CV standards.

The liver and fiducial board have been scanned using a CT scanner and processed to create a volumetric model of the liver and the fidicual board.  Both of these models are available, in vtk and ply formats, in the **ct scans** subfolder.  These models can be used to aid in an initial alignment between your 3D surface reconstruction and the CT scan as well as to report measures of surface registration error (comparing your registered reconstruction to the CT scan of the liver).


# Computing error metrics from transformation estimate
## Computing the angle of rotation

Assume the input is a 4x4 matrix representing the rigid transformation estimate between the current frame and the keyframe (frame 1), <img src="https://render.githubusercontent.com/render/math?math=T">.

The rotational component of <img src="https://render.githubusercontent.com/render/math?math=T"> can be represented as a quaternion, <img src="https://render.githubusercontent.com/render/math?math=T_q">, and the angle of rotation can then be described as

angle of rotation = <img src="https://render.githubusercontent.com/render/math?math=2 * \arccos(T_q(1))">

where <img src="https://render.githubusercontent.com/render/math?math=T_q(1)"> is the first element of the vector representation of the quaternion.

##### Matlab example: Angle of rotation computation
```Matlab
% Assume 4x4 rigid transformation in variable Ta

% Compute quaternion representation of the rotation portion of transformation Ta
dq = qm(Ta);

% Compute the angle of rotation from dq
angle_of_rotation = 2 * acosd(dq(1));

function q = qm(M)
% QM  Quaternion from affine matrix.
%   Q = QM(M)

	q = zeros(1,7);
	q(1,1) = 0.5*sqrt(trace(M));
	q(1,2) = (M(3,2) - M(2,3))/(4*q(1));
	q(1,3) = (M(1,3) - M(3,1))/(4*q(1));
	q(1,4) = (M(2,1) - M(1,2))/(4*q(1));
	q(1,5) = M(1,4);
	q(1,6) = M(2,4);
	q(1,7) = M(3,4);
end
```

## Computing the translation magnitude

Before computing the translation magnitude the center of rotation has to be removed from the transformation estimate.  Then the magnitude of the translation component can be shown as the L2-norm of the translation component of the 4x4 transformation matrix.

##### Matlab example: Translation magnitude computation
```Matlab
% Assume 4x4 rigid transformation in variable Ta
% Assume center of rotation 1x3 vector for current frame in decen

% Compute translation-only 4x4 transformation matrix from decen
Ddecen = [eye(3) -decen; 0 0 0 1];

% Compute rotation-only 4x4 transformation matrix from Ta
R = Ta;
R(1:3, 4) = [0 0 0]';

% Compute residual translation, dVec, after removing center of rotation
Ddelta = Ta * inv(inv(Ddecen) * R * Ddecen);
dVec = Ddelta(1:3, 4);

% Compute magnitude of translation component
dmag = sqrt(sum(dVec .* dVec)) * S;
```

The center of rotation values are stored in the ``center\_of\_rotation'' folder and are labelled cor\_1 - cor\_30.  In general, cor\_n should be used in the computation of the translation component of the transformation estimate between frames n and n+1.

# Citation

Please site our paper whenever presenting work that makes use of this dataset.

```
@article{Speers2018,
author = {Speers, Andrew D and Ma, Burton and Jarnagin, William R and Himidan, Sharifa and Simpson, Amber L and Wildes, Richard P},
title = {Fast and accurate vision-based stereo reconstruction and motion estimation for image-guided liver surgery},
journal = {Medical Image Computing and Computer Assisted Interventions Workshop (MICCAIW) on Augmented Environments for Computer-Assisted Interventions (AE-CAI) Published in \em{Healthcare Technology Letters}},
year = {2018},
volume = {5},
number = {5},
pages = {208--214}
}
```
