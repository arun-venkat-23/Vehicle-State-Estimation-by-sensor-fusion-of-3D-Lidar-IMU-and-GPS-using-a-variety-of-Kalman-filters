# Vehicle State Estimation by sensor fusion

<p> Dive into the realm of advanced sensor fusion as we explore the integration of IMU, GPS, and Lidar through the sophisticated lens of an Extended Kalman Filter. Our journey commences with the meticulous conversion of the 3D Carla dataset from 'pkl' to 'csv' format, courtesy of a meticulously crafted data conversion script. <br>

Within this technological tapestry, distinct datasets unfold – the ground truth data, a repository of timestamps, positions, velocities, accelerations, rotations, and rotational rates; the IMU data, encapsulating accelerations and gyroscopes; the Lidar data, articulating a spatial narrative with X, Y, and Z position values; and the GPS data, unveiling the coordinates in Easting, Northing, and Up. <br>

This amalgamation of sensors creates a comprehensive panorama, offering a nuanced understanding of a vehicle's state. It transcends numerical data, delving into the intricacies of motion dynamics. Join us on this expedition where algorithms seamlessly merge diverse data streams, demystifying the complexities of a vehicle's dynamic profile. In this narrative, technology and innovation converge to transform raw data into the eloquence of motion – a captivating symphony of precision and advancement. Welcome to the frontier where sensor fusion unfolds the poetry of vehicular dynamics. </p>

## Reading data from .pkl files

``` DataConversion.py``` - This Python code utilizes the Pandas library to transform Lidar data stored in the variable lidar.data into an Excel spreadsheet. The script begins by accessing the Lidar data and storing it in the variable a. Subsequently, it creates a Pandas DataFrame (df) with columns labeled 'X', 'Y', and 'Z', representing the dimensions of the Lidar data. The code then establishes an Excel writer (datatoexcel) using Pandas, specifying the filename as "Lidar.xlsx" and utilizing the 'xlsxwriter' engine. The Lidar data DataFrame is written to the Excel file under the sheet named 'Sheet1'. Finally, the Excel file is saved, resulting in a structured spreadsheet containing Lidar data with columns denoting X, Y, and Z coordinates.

## Quaternion Class

This Python code defines a Quaternion class and associated functions for quaternion operations, primarily used in 3D rotations. The skew_symmetric function constructs a skew-symmetric matrix from a 3D vector. The Quaternion class enables quaternion initialization through direct specification of components, axis-angle representation, or Euler angles. Notably, it ensures consistency by allowing only one of axis-angle or Euler angles to be specified. The class provides methods for converting quaternions to rotation matrices, Euler angles, NumPy arrays, and supports quaternion multiplication. Quaternion multiplication involves creating a 4x4 matrix and performing the multiplication, with options to output as a NumPy array or a new Quaternion object. This code serves as a robust and versatile implementation for working with quaternions, offering functionalities essential for applications involving 3D rotations and sensor fusion.

## Reading data from .csv files

This Python code snippet is dedicated to the retrieval and adjustment of data from four distinct CSV files: "GroundTruth_Data.csv," "GPS_Data.csv," "Lidar_Data.csv," and "IMU_Data.csv." Each file contains columns representing different facets of a system's state, such as position, velocity, acceleration, Euler angles, rotational rate, and rotational acceleration. The code employs NumPy's genfromtxt function to read the data from each CSV file, assuming a comma as the delimiter. Subsequently, it modifies the first entry in the Timestep column of each dataset to a specific value (2.055). This adjustment likely aligns the starting time reference across the datasets, ensuring coherence in the temporal domain. The resulting NumPy arrays, namely gt, gps, lidar, and imu, serve as organized data structures for further analysis or integration into a broader computational system.

## Transformations

The LIDAR frame is not the same as the frame shared by the IMU and the GPS. So, we transform the LIDAR data to the IMU and GPS frame using extrinsic calibration rotation matrix C_li and translation vector t_i_li. We assume the euler transformation is takes place in the ZYX order. This gives us the rotation matrix which would transform the Lidar values to the IMU+GPS frame.

![image](https://github.com/arun-venkat-23/Vehicle-State-Estimation-by-sensor-fusion-of-3D-Lidar-IMU-and-GPS-using-a-variety-of-Kalman-filters/assets/137104589/e3234159-98e4-4f62-a21f-4ff8b6ed9f6f)

Computing correct calibration rotation matrix corresponding to Euler RPY angles ( gamma = 0.05, beta = 0.05, alpha = 0.1). Since the Lidar is placed at a different location from the IMU+GPS combo, we need to translate the Lidar readings to the proper frame of reference to ensure all readings and estimates are computed with a single frame of reference. Thus, we calculate the translation matrix, which turns out to be (x = 0.5, y = 0.1, z = 0.5).
