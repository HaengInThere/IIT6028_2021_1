# 계산영상시스템 Homework #2 : Eulerian Video Magnification
# 전기전자공학부 2020314086 이정행


## Initials and color transformation + Laplacian Pyramid 
Video read, convert to double and normalize. Convert to YIQ color space.
Make Laplacian pyramids with decomposition level 4.
```matlab
%% Initials and Color Transformation

clc; clear all;

data_path = 'G:/수업/수강/[2-1]계산영상시스템/hw/2/hw2/data/';
file_name = 'my.mp4';
decomp_level = 4;
video = VideoReader([data_path  file_name]);

double_video = zeros(video.Height, video.Width, 3, video.NumFrames);
pyramids_video = {};
[H, W, c] = size(squeeze(double_video(:,:,:,1)));

for frm = 1:video.NumFrames
    pyramids = {};
    laplacian_pyramids = {};
    frame = read(video, frm);
    double_frame = normalize(double(frame));

    YIQ_frame = rgb2ntsc(double_frame);

    pyramids{1} = YIQ_frame;
    for i = 1:decomp_level
        pyramids{i+1} = impyramid(pyramids{i}, 'reduce');
    end

    for j = 1:decomp_level - 1
        laplacian_pyramids{j} = pyramids{j} - imresize3(pyramids{j+1}, [round(H / (2)^(j-1)), round(W / (2)^(j-1)), c]);        
    end
    laplacian_pyramids{decomp_level} = pyramids{decomp_level};

    pyramids_video{frm} = laplacian_pyramids;
end
```
## Dimension manipulation for intuitive computation

```matlab
%% dimension changing

level_cell = {};
for frm = 1:video.NumFrames
    
    curr_pyramid = pyramids_video{frm};
    for j = 1:decomp_level
       level_cell{j}{frm} = curr_pyramid{j};
    end
end

%% video cell to each pixel column

[H,W,c] = size(level_cell{1}{1});
pixel_set_1 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3
            for frm = 1:video.NumFrames
                pixel_set_1(frm, j,i,c) = level_cell{1}{frm}(j,i,c);
            end
        end
    end
end

[H,W,c] = size(level_cell{2}{1});
pixel_set_2 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3
            for frm = 1:video.NumFrames
                pixel_set_2(frm, j,i,c) = level_cell{2}{frm}(j,i,c);
            end
        end
    end
end


[H,W,c] = size(level_cell{3}{1});
pixel_set_3 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3
            for frm = 1:video.NumFrames
                pixel_set_3(frm, j,i,c) = level_cell{3}{frm}(j,i,c);
            end
        end
    end
end

[H,W,c] = size(level_cell{4}{1});
pixel_set_4 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3
            for frm = 1:video.NumFrames
                pixel_set_4(frm, j,i,c) = level_cell{4}{frm}(j,i,c);
            end
        end
    end
end

```

## Temporal Filtering
With pre-saved pixel columns(.mat), apply temporal filtering with fft func.
```matlab
%% pixel read 

pixel_set_1 = load('pixel_set_1').pixel_set_1;
pixel_set_2 = load('pixel_set_2').pixel_set_2;
pixel_set_3 = load('pixel_set_3').pixel_set_3;
pixel_set_4 = load('pixel_set_4').pixel_set_4;

%% filtering

Fs = video.FrameRate;
Hd = butterworthBandpassFilter(Fs, 256, 0.8, 1);

HD_f = freqz(Hd, video.NumFrames);

[H,W,c] = size(level_cell{1}{1});
filtered_1 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3     
            fft_1 = fft(pixel_set_1(:,j,i,c));
            filtered_1(:,j,i,c) = abs(ifft(fft_1 .* HD_f));
        end
    end
end

[H,W,c] = size(level_cell{2}{1});
filtered_2 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3     
            fft_2 = fft(pixel_set_2(:,j,i,c));
            filtered_2(:,j,i,c) = abs(ifft(fft_2 .* HD_f));
        end
    end
end

[H,W,c] = size(level_cell{3}{1});
filtered_3 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3     
            fft_3 = fft(pixel_set_3(:,j,i,c));
            filtered_3(:,j,i,c) = abs(ifft(fft_3 .* HD_f));
        end
    end
end


[H,W,c] = size(level_cell{4}{1});
filtered_4 = zeros(video.NumFrames, H, W, c);
for j = 1:H
    for i = 1:W
        for c = 1:3     
            fft_4 = fft(pixel_set_4(:,j,i,c));
            filtered_4(:,j,i,c) = abs(ifft(fft_4 .* HD_f));
        end
    end
end

```
## Extracting the frequency band of interest
Filter visualization results

<p align='center'>
  <img src='./Figs/hw2/spec_1.png' width="240px">
  <img src='./Figs/hw2/spec_2.png' width="240px">
</p>


## Image reconstruction 
Reconstruct the images with pyramids and save video.

```matlab
%% image reconstruction

[H,W,c] = size(level_cell{1}{1});

alpha_1 = 10;
alpha_2 = 10;
alpha_3 = 10;
alpha_4 = 10;
video = VideoReader([data_path  file_name]);
recon_frames = [];
for i =1:video.NumFrames
    frame = read(video, i);
    double_frame = normalize(double(frame));

    YIQ_frame = rgb2ntsc(double_frame);
    recon = YIQ_frame + ...
            squeeze(filtered_1(frm, :,:,:)) * alpha_1 + ...
            imresize3(squeeze(filtered_2(frm,:,:,:) * alpha_2), [H,W,c]) + ...
            imresize3(squeeze(filtered_3(frm,:,:,:) * alpha_3), [H,W,c]) + ...
            imresize3(squeeze(filtered_4(frm,:,:,:) * alpha_4), [H,W,c]);
    recon_frames = cat(4, recon_frames ,  ntsc2rgb(cat(3, recon(:,:,1), recon(:,:,2), recon(:,:,3))));
end


%% Write video
output_video = VideoWriter('output_my_2.avi');
output_video.FrameRate = 30;
open(output_video);

for i=1:video.NumFrames
    new_frame = squeeze(recon_frames(:, :, :, i));
    writeVideo(output_video, new_frame);
end

close(output_video);
```

<p align='center'>
  <img src='./Figs/hw2/baby.gif' width="360px">
  <img src='./Figs/hw2/face.gif' width="360px">
</p>

## Extra Credit: Capture and motion-magnify your own video.
### 2 Results according to various filters.

<p align='center'>
  <img src='./Figs/hw2/my_1.gif' width="360px">
  <img src='./Figs/hw2/my_2.gif' width="360px">
</p>

