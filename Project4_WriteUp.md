# Project 4: Building an Estimator Write Up #
###### Implemented by Siddhartha Kumar ####


#### 1. Sensor Noise:
   - Processed the logs GPS X data and Accelerometer X data and calculated the Gps and Accel Stddev
   - MeasuredStdDev_GPSPosXY = 0.69
   - MeasuredStdDev_AccelXY = 0.4792
   - Results:
   ```
   PASS: ABS(Quad.GPS.X-Quad.Pos.X) was less than MeasuredStdDev_GPSPosXY for 68% of the time
   PASS: ABS(Quad.IMU.AX-0.000000) was less than MeasuredStdDev_AccelXY for 67% of the time
   ```


#### 2. Attitude Estimation:
   - Implemented the Complementary filter for the `UpdateFromIMU()` function
   - To improve the performance of the IMU update estimator the attitude was converted to a quaternion and integrated with the gyro rates in the inertial frame
   - The yaw state was normalized so as to handle angle wrapping correctly
   - Simulation Results:
   ```
   Simulation #11 (../config/07_AttitudeEstimation.txt)
   PASS: ABS(Quad.Est.E.MaxEuler) was less than 0.100000 for at least 3.000000 seconds
   ```
   
#### 3.Prediction Step:
   - The `PredictState` function was implemented to provide the predicted mean from the IMU measurements
   - The `ekfState` vector was update :
      - Predicited X,Y,Z were updated by the integrated velocities
      - Velocity X,Y,Z were update by the integrated accel measurements in the body frame
      - Yaw was updated directly as it was already calculated in the `UpdateFromIMU()` step
   - Running Simulation scenario 8 the drone followed the Square pattern with a slow drift in the estimate
   - The Jacobian was implemented in `GetRbgPrime()` referencing the equations in section 7.2.
   - Then the `Predict()` Function was a simple matter of implementing the covariance calculation and incorporating the process noise.
   - The New state and co-varaince were then updated
   - Running Scenario 9 the XY position and velocity standard deviations were modified to better capture the drift
      - QPosXYStd = .03
      - QVelXYStd = .18
   
#### 4. Magnetometer Update:
   - Running Scenario 10 the YawStd deviation was calibrated to capture the drift, to do this better I modified the script to keep the drone stationary and observe the drift in yaw over the 20 second period
     - QYawStd = .08
   - Then the `UpdateFromMag()` function was implemented, In this case the sensor model was a simple 7x1 vector that only provided us a yaw update.
   - To ensure there was no wrapping errors in our estimate the difference between the measurement and the predicited Yaw was checked to normalize the Predict measurement so that it is handled correclty in the `Update()` step

   
#### 5. GPS Update:
   - Implemented the `UpdateFromGps()` function using the measurement model as a simple 6x7 "identity" matrix (main diagonal = 1)
   - The process noise model seemed to capture the error seen from running Scenario 11
   - Scenario 11 was run with the implemented Estimator and non-ideal sensors
      - Results:
      ```
      Simulation #19 (../config/11_GPSUpdate.txt)
      PASS: ABS(Quad.Est.E.Pos) was less than 1.000000 for at least 20.000000 seconds
      ```

#### 6.Adding My Control (From P3):
   - Replaced the QuadControl.cpp and QuadControlParams.cpp
   - Tweaked the gains by about 20 - 30% and saw an improvement in performance
   - An issue was discovered with the controllers handling of wrapping of the Yaw angle and was updated
   - Running Scenario 11 the drone follows the square with a less than 1m position error.