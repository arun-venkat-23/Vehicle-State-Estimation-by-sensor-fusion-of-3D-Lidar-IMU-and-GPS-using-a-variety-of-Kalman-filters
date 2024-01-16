# Vehicle State Estimation by sensor fusion

<p> Dive into the realm of advanced sensor fusion as we explore the integration of IMU, GPS, and Lidar through the sophisticated lens of an Extended Kalman Filter. Our journey commences with the meticulous conversion of the 3D Carla dataset from 'pkl' to 'csv' format, courtesy of a meticulously crafted data conversion script. <br>

Within this technological tapestry, distinct datasets unfold – the ground truth data, a repository of timestamps, positions, velocities, accelerations, rotations, and rotational rates; the IMU data, encapsulating accelerations and gyroscopes; the Lidar data, articulating a spatial narrative with X, Y, and Z position values; and the GPS data, unveiling the coordinates in Easting, Northing, and Up. <br>

This amalgamation of sensors creates a comprehensive panorama, offering a nuanced understanding of a vehicle's state. It transcends numerical data, delving into the intricacies of motion dynamics. Join us on this expedition where algorithms seamlessly merge diverse data streams, demystifying the complexities of a vehicle's dynamic profile. In this narrative, technology and innovation converge to transform raw data into the eloquence of motion – a captivating symphony of precision and advancement. Welcome to the frontier where sensor fusion unfolds the poetry of vehicular dynamics. </p>

## Reading data from .pkl files
<p>
``` DataConversion.py``` - This Python code utilizes the Pandas library to transform Lidar data stored in the variable lidar.data into an Excel spreadsheet. The script begins by accessing the Lidar data and storing it in the variable a. Subsequently, it creates a Pandas DataFrame (df) with columns labeled 'X', 'Y', and 'Z', representing the dimensions of the Lidar data. The code then establishes an Excel writer (datatoexcel) using Pandas, specifying the filename as "Lidar.xlsx" and utilizing the 'xlsxwriter' engine. The Lidar data DataFrame is written to the Excel file under the sheet named 'Sheet1'. Finally, the Excel file is saved, resulting in a structured spreadsheet containing Lidar data with columns denoting X, Y, and Z coordinates. </p>

## Quaternion Class

<p> This Python code defines a Quaternion class and associated functions for quaternion operations, primarily used in 3D rotations. The skew_symmetric function constructs a skew-symmetric matrix from a 3D vector. The Quaternion class enables quaternion initialization through direct specification of components, axis-angle representation, or Euler angles. Notably, it ensures consistency by allowing only one of axis-angle or Euler angles to be specified. The class provides methods for converting quaternions to rotation matrices, Euler angles, NumPy arrays, and supports quaternion multiplication. Quaternion multiplication involves creating a 4x4 matrix and performing the multiplication, with options to output as a NumPy array or a new Quaternion object. This code serves as a robust and versatile implementation for working with quaternions, offering functionalities essential for applications involving 3D rotations and sensor fusion. </p>

## Reading data from .csv files

<p> This Python code snippet is dedicated to the retrieval and adjustment of data from four distinct CSV files: "GroundTruth_Data.csv," "GPS_Data.csv," "Lidar_Data.csv," and "IMU_Data.csv." Each file contains columns representing different facets of a system's state, such as position, velocity, acceleration, Euler angles, rotational rate, and rotational acceleration. The code employs NumPy's genfromtxt function to read the data from each CSV file, assuming a comma as the delimiter. Subsequently, it modifies the first entry in the Timestep column of each dataset to a specific value (2.055). This adjustment likely aligns the starting time reference across the datasets, ensuring coherence in the temporal domain. The resulting NumPy arrays, namely gt, gps, lidar, and imu, serve as organized data structures for further analysis or integration into a broader computational system. </p>

## Transformations

<p> The LIDAR frame is not the same as the frame shared by the IMU and the GPS. So, we transform the LIDAR data to the IMU and GPS frame using extrinsic calibration rotation matrix C_li and translation vector t_i_li. We assume the euler transformation is takes place in the ZYX order. This gives us the rotation matrix which would transform the Lidar values to the IMU+GPS frame.

![image](https://github.com/arun-venkat-23/Vehicle-State-Estimation-by-sensor-fusion-of-3D-Lidar-IMU-and-GPS-using-a-variety-of-Kalman-filters/assets/137104589/e3234159-98e4-4f62-a21f-4ff8b6ed9f6f)

Computing correct calibration rotation matrix corresponding to Euler RPY angles ( gamma = 0.05, beta = 0.05, alpha = 0.1). Since the Lidar is placed at a different location from the IMU+GPS combo, we need to translate the Lidar readings to the proper frame of reference to ensure all readings and estimates are computed with a single frame of reference. Thus, we calculate the translation matrix, which turns out to be (x = 0.5, y = 0.1, z = 0.5). </p>

## Sensor Covariance

<p> Variance is computed by taking the difference between the groundtruth values and the respective X,Y,Z of GPS and Lidar. The variance of this difference would give me the respective variances in the X,Y and Z directions. Average of the three variances would provide the variance of that particular sensor. When using an accelerometer or gyroscope, the variance is calculated by averaging Ax, Ay, and Az, subtracting the average from each value to obtain the difference, and then computing the mean of the squared difference to provide the variance matrix. </p>

## Variable Initialization

<p> This Python code initializes variables and matrices for a sensor fusion algorithm, likely involving an Extended Kalman Filter (EKF) for state estimation in a dynamic system. The g vector represents gravity, and matrices Lj and Hj are Jacobians used in the motion and measurement models, respectively. The matrices p, v, a, e, er, q, x_state, and p_cov are state variables, tracking position, velocity, acceleration, Euler angles, rotational rates, quaternions, state vectors, and covariance matrices at each timestep. The initial states are set using ground truth data (gt). The system's initial position, velocity, and orientation are initialized, while acceleration and rotational acceleration are commented out, possibly indicating they are not used initially. The code suggests the incorporation of GPS and Lidar measurements (gps_i and lidar_i variables) into the sensor fusion process. This initialization lays the foundation for subsequent state estimation and sensor fusion computations in the algorithm. </p>

## Kalman Filtering

<p> The Python function ekf represents an Extended Kalman Filter (EKF) correction step for sensor fusion, commonly used in state estimation for dynamic systems. The function takes as input the sensor variance (sensor_var), the covariance matrix of the predicted state (p_cov), the measurement vector (y_k), and the predicted state vector (x_state). <br>

The Kalman Gain (K) is computed based on the predicted state covariance matrix, measurement Jacobian (Hj), and the sensor variance. The error (e) between the measurement and the predicted state is then calculated. The predicted state is corrected by incorporating the Kalman Gain-weighted error, updating the estimates for position (p), velocity (v), acceleration (a), Euler angles (e), and rotational rates (er). The corrected state vector is then updated. <br>

Finally, the function computes the corrected covariance matrix (p_cov_hat) by adjusting the predicted state covariance using the Kalman Gain and measurement Jacobian. The corrected state vector and covariance matrix are returned, representing an updated estimate based on the measurement input.<br>

Notably, the code also involves quaternion operations using the Quaternion class for representing and manipulating orientations. The corrected Euler angles (e) are obtained from the quaternion representation of the corrected orientation. </p>

## Measurement Model

```State Transition Matrix (F)```: The matrix F represents the linearized state transition model that predicts the next state based on the current state. It accounts for the motion model, updating position, velocity, acceleration, and other state variables. <br>
```Acceleration Calculation (temp)```: The code computes the linear acceleration due to motion using the previous orientation, previous IMU acceleration, and gravitational acceleration. <br>
```Process Noise Covariance Matrix (Q)```: This matrix represents the covariance of the process noise affecting the system. It is scaled by the variance of linear acceleration (var_imu_f) and angular velocity (var_imu_w), and then scaled by the square of the time difference. <br>
```State Prediction```: The state is predicted for the next timestep using the state transition matrix and the previous state. <br>
```Covariance Prediction```: The covariance matrix is predicted for the next timestep using the state transition matrix, the previous covariance matrix, and the process noise covariance matrix. <br>
```Correction Step (GPS and LIDAR)```: If GPS or LIDAR measurements are available within a small time window (0.01 seconds), an Extended Kalman Filter correction is applied to update the state estimate and covariance matrix based on the sensor measurements. <br>

The main purpose of this code is to propagate the state estimate and covariance over time using IMU measurements, while also incorporating corrections from GPS and LIDAR measurements when available. The EKF helps refine the state estimate by fusing information from different sensors and accounting for uncertainties.

## Result

#### With Lidar and GPS
![image](https://github.com/arun-venkat-23/Vehicle-State-Estimation-by-sensor-fusion-of-3D-Lidar-IMU-and-GPS-using-a-variety-of-Kalman-filters/assets/137104589/f14e08f0-4e36-4f8a-b363-1f04ea613129)

#### With only Lidar
![image](https://github.com/arun-venkat-23/Vehicle-State-Estimation-by-sensor-fusion-of-3D-Lidar-IMU-and-GPS-using-a-variety-of-Kalman-filters/assets/137104589/798dce6d-be5c-4a06-86a6-f1216763907d)

#### Error plots - With Lidar and GPS
![image](https://github.com/arun-venkat-23/Vehicle-State-Estimation-by-sensor-fusion-of-3D-Lidar-IMU-and-GPS-using-a-variety-of-Kalman-filters/assets/137104589/6a0fed08-563d-4dcc-8d84-a9346e2a8558)

#### Error plots - With Lidar
![image](https://github.com/arun-venkat-23/Vehicle-State-Estimation-by-sensor-fusion-of-3D-Lidar-IMU-and-GPS-using-a-variety-of-Kalman-filters/assets/137104589/138797a9-3700-419a-a6c1-ee94355b855f)



