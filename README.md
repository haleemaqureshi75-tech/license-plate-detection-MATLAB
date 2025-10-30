# license-plate-detection-MATLAB
Detects and extracts text from vehicle license plates using image processing and OCR in MATLAB
Clear all
clc
% Step 1: Read and Display Image
image = imread('car_image.jpg'); % Replace 'car_image.jpg' with your image name
imshow(image); title('Original Image');
% Step 2: Convert to Grayscale
grayImage = rgb2gray(image);
% Step 3: Apply Gaussian Blur to Reduce Noise
blurredImage = imgaussfilt(grayImage, 2); % Gaussian filter with sigma = 2
% Step 4: Edge Detection
edgeImage = edge(blurredImage, 'Canny');
imshow(edgeImage); title('Edge Detection');
% Step 5: Find Contours
% MATLAB uses regionprops for finding regions and bounding boxes
se = strel('rectangle', [5, 5]);
dilatedImage = imdilate(edgeImage, se);
[labeledImage, numRegions] = bwlabel(dilatedImage);
props = regionprops(labeledImage, 'BoundingBox', 'Area');
% Step 6: Filter Potential License Plate Regions
imshow(image); title('Potential License Plates');
hold on;
for i = 1:numRegions
    boundingBox = props(i).BoundingBox;
    area = props(i).Area;
    aspectRatio = boundingBox(3) / boundingBox(4); % width / height
    if aspectRatio > 2.0 && aspectRatio < 6.0 && area > 1000 && area < 20000
        % Highlight the detected region
        rectangle('Position', boundingBox, 'EdgeColor', 'g', 'LineWidth', 2);
        % Step 7: Crop and Process License Plate Region
        croppedPlate = imcrop(grayImage, boundingBox);
        binaryPlate = imbinarize(croppedPlate, 'adaptive');
        imshow(binaryPlate); title('Binary License Plate');
        % Step 8: OCR to Extract Text
        ocrResults = ocr(binaryPlate, 'TextLayout', 'Block');
        detectedText = regexprep(ocrResults.Text, '\W', ''); % Remove non-alphanumeric characters
        disp(['Detected License Plate Text: ', strtrim(detectedText)]);
        % Display the result
        imshow(image); hold on;
        rectangle('Position', boundingBox, 'EdgeColor', 'g', 'LineWidth', 2);
        text(boundingBox(1), boundingBox(2) - 10, detectedText, 'Color', 'yellow', 'FontSize', 12);
        return; % Exit once a license plate is detected
    end
end
disp('No license plate detected.');

