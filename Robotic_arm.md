% Robotic Arm with Color Detection - Pick and Place
% 3 Servos: S1-Base, S2-Shoulder, S3-Gripper
clear all;
close all;
clc;

%% Initialize Arduino Connection
fprintf('Connecting to Arduino...\n');
a = arduino('COM6', 'Uno'); % Change COM port as needed

% Create servo objects
s1_base = servo(a, 'D9');      % Base servo on pin 9
s2_shoulder = servo(a, 'D10'); % Shoulder servo on pin 10
s3_gripper = servo(a, 'D11');  % Gripper servo on pin 11

fprintf('Arduino connected successfully!\n\n');

%% Set Initial Positions
fprintf('Setting initial positions...\n');
writePosition(s1_base, 60/180);      % Base at 60 degrees
writePosition(s2_shoulder, 60/180);  % Shoulder at 60 degrees
writePosition(s3_gripper, 100/180);  % Gripper at 100 degrees (open)
pause(2); % Wait for servos to reach position
fprintf('Initial positions set.\n\n');

%% Initialize Webcam (DroidCam - Mobile as Webcam)
fprintf('Initializing DroidCam mobile webcam...\n');
video = videoinput('winvideo', 1);
set(video, 'ReturnedColorSpace', 'RGB');
preview(video);
fprintf('Webcam initialized and preview started.\n\n');

%% Color Detection Phase
fprintf('Starting color detection...\n');
fprintf('Place the object at the fixed position.\n\n');

detectedColor = ''; % Variable to store detected color

% Preview and detect color
while isempty(detectedColor)
    img = getsnapshot(video);
    % Display the image for detection
    figure(1);
    imshow(img);
    title('Detecting Color... Place object in center');
    drawnow;
    
    % Define the region of interest (center of image)
    [rows, cols, ~] = size(img);
    roi_size = 100; % Size of ROI
    row_start = round(rows/2 - roi_size/2);
    row_end = round(rows/2 + roi_size/2);
    col_start = round(cols/2 - roi_size/2);
    col_end = round(cols/2 + roi_size/2);
    
    % Extract ROI
    roi = img(row_start:row_end, col_start:col_end, :);
    
    % Convert to HSV for better color detection
    hsv_roi = rgb2hsv(roi);
    
    % Define color ranges in HSV
    % Red color range (HSV: Hue 0-10 or 160-180, Sat 0.4-1, Val 0.4-1)
    red_mask1 = (hsv_roi(:,:,1) >= 0 & hsv_roi(:,:,1) <= 0.05) & ...
                (hsv_roi(:,:,2) >= 0.4) & (hsv_roi(:,:,3) >= 0.4);
    red_mask2 = (hsv_roi(:,:,1) >= 0.95 & hsv_roi(:,:,1) <= 1) & ...
                (hsv_roi(:,:,2) >= 0.4) & (hsv_roi(:,:,3) >= 0.4);
    red_mask = red_mask1 | red_mask2;
    
    % Green color range (HSV: Hue 0.25-0.45, Sat 0.4-1, Val 0.2-1)
    green_mask = (hsv_roi(:,:,1) >= 0.25 & hsv_roi(:,:,1) <= 0.45) & ...
                 (hsv_roi(:,:,2) >= 0.4) & (hsv_roi(:,:,3) >= 0.2);
    
    % Count pixels
    red_pixels = sum(red_mask(:));
    green_pixels = sum(green_mask(:));
    
    % Display detection info
    text_str = sprintf('Red pixels: %d | Green pixels: %d', red_pixels, green_pixels);
    text(10, 30, text_str, 'Color', 'yellow', 'FontSize', 12, 'BackgroundColor', 'black');
    
    % Threshold for detection (adjust if needed)
    threshold = 500;
    
    if red_pixels > threshold && red_pixels > green_pixels
        detectedColor = 'red';
        fprintf('RED color detected!\n');
        % Capture screenshot
        screenshot = img;
        imwrite(screenshot, 'detected_object.jpg');
        fprintf('Screenshot saved as "detected_object.jpg"\n\n');
        pause(1);
        break;
    elseif green_pixels > threshold && green_pixels > red_pixels
        detectedColor = 'green';
        fprintf('GREEN color detected!\n');
        % Capture screenshot
        screenshot = img;
        imwrite(screenshot, 'detected_object.jpg');
        fprintf('Screenshot saved as "detected_object.jpg"\n\n');
        pause(1);
        break;
    end
    
    pause(0.1); % Small delay for display update
end

% Close preview and stop video input
closepreview(video);
stop(video);
delete(video);
clear video;
fprintf('Webcam preview closed.\n\n');

% Display the captured screenshot
figure('Name', 'Detected Object', 'NumberTitle', 'off');
imshow(screenshot);
title(['Detected Color: ' upper(detectedColor)]);

%% Robotic Arm Movement Sequence
fprintf('Starting robotic arm movement...\n\n');

% Step 2: Base moves from 60 to 90 degrees
fprintf('Step 1: Moving base from 60 to 90 degrees...\n');
moveServo(s1_base, 60, 90, 'Base');

% Step 3: Shoulder moves from 60 to 100 degrees
fprintf('Step 2: Moving shoulder from 60 to 100 degrees...\n');
moveServo(s2_shoulder, 60, 100, 'Shoulder');

% Step 4: Gripper closes (100 to 170 degrees) - Pick object
fprintf('Step 3: Closing gripper (100 to 170 degrees) - Picking object...\n');
moveServo(s3_gripper, 100, 170, 'Gripper');
pause(1);
fprintf('Object picked!\n\n');

% Step 5: Shoulder moves back (100 to 60 degrees)
fprintf('Step 4: Moving shoulder back (100 to 60 degrees)...\n');
moveServo(s2_shoulder, 100, 60, 'Shoulder');

% Step 6: Base moves based on detected color
if strcmp(detectedColor, 'red')
    fprintf('Step 5: RED detected - Moving base from 90 to 40 degrees...\n');
    moveServo(s1_base, 90, 40, 'Base');
    fprintf('Object placed at RED zone (40 degrees)!\n\n');
elseif strcmp(detectedColor, 'green')
    fprintf('Step 5: GREEN detected - Moving base from 90 to 20 degrees...\n');
    moveServo(s1_base, 90, 20, 'Base');
    fprintf('Object placed at GREEN zone (20 degrees)!\n\n');
end

% Step 7: Gripper opens (170 to 100 degrees) - Release object
fprintf('Step 6: Opening gripper (170 to 100 degrees) - Releasing object...\n');
moveServo(s3_gripper, 170, 100, 'Gripper');
pause(1);
fprintf('Object released!\n\n');

fprintf('==============================================\n');
fprintf('Pick and place operation completed successfully!\n');
fprintf('Detected color: %s\n', upper(detectedColor));
fprintf('==============================================\n');

% Clean up
clear s1_base s2_shoulder s3_gripper a;

%% Helper Function: Smooth Servo Movement
function moveServo(servoObj, startAngle, endAngle, servoName)
    stepSize = 2; % Degrees per step (smaller = smoother)
    delayTime = 0.05; % Seconds between steps
    
    if startAngle < endAngle
        % Moving forward
        for angle = startAngle:stepSize:endAngle
            writePosition(servoObj, angle/180);
            fprintf('  %s: %d degrees\n', servoName, angle);
            pause(delayTime);
        end
    else
        % Moving backward
        for angle = startAngle:-stepSize:endAngle
            writePosition(servoObj, angle/180);
            fprintf('  %s: %d degrees\n', servoName, angle);
            pause(delayTime);
        end
    end
    
    % Ensure final position is reached
    writePosition(servoObj, endAngle/180);
    fprintf('  %s reached %d degrees\n\n', servoName, endAngle);
    pause(0.5);
end