# 계산영상시스템 Homework #1 : Demosaic

## Implementation a basic image processing pipeline

## Initials
Image read, convert to 2D-double array

```matlab
clc; clear all;

img_cr2 = imread('G:\수업\수강\[2-1]계산영상시스템\hw\1\assign1\data\banana_slug.cr2');
img_tiff = imread('G:\수업\수강\[2-1]계산영상시스템\hw\1\assign1\data\banana_slug.tiff');

% image size 
[H, W] = size(img_tiff);

class(img_tiff)
double_img = double(img_tiff);
```
## Linearization
Linear tranformation for img and clipping
```matlab
% Linearization into range [0,1] using linear transformation
norm_img = (double_img/(15000 - 2047)) - (2047 / (15000-2047));
% clipping
norm_img(norm_img<0) = 0;
norm_img(norm_img>1) = 1;
```

## Identifying the correct Bayer pattern
Comparing the four cases, it was determined that the given image has a pattern of rggb.

```matlab
%% rggb
im_r = norm_img(1:2:end, 1:2:end);
im_b = norm_img(2:2:end, 2:2:end);
im_g_1 = norm_img(1:2:end, 2:2:end);
im_g_2 = norm_img(2:2:end, 1:2:end);

im_g = (im_g_1 + im_g_2) /2 ;
im_rgb = cat(3, im_r, im_g, im_b);
image(im_rgb)
```

## White balancing
Two cases of white balancing : Gray world assumption & White world assumption

```matlab
%% white balancing
% gray world assumption

gray_bal_r = mean(im_g(:)) / mean(im_r(:)) * im_r;
gray_bal_g = im_g;
gray_bal_b = mean(im_g(:)) / mean(im_b(:)) * im_b;

gray_bal_im_rgb = cat(3, gray_bal_r, gray_bal_g, gray_bal_b);
figure()
image(gray_bal_im_rgb)

% white world assumption

white_bal_r = max(im_g(:)) / max(im_r(:)) * im_r;
white_bal_g = im_g ;
white_bal_b = max(im_g(:)) / max(im_b(:)) * im_b;

white_bal_im_rgb = cat(3, white_bal_r, white_bal_g, white_bal_b);
figure()
image(white_bal_im_rgb)
```

## Demosiacing
Bilinear interpolation for demosaicing. (using interp2 function in MatLab)

```matlab
%% demosaic

gray_interpol_r = interp2(gray_bal_r);
gray_interpol_g = interp2(gray_bal_g);
gray_interpol_b = interp2(gray_bal_g);
gray_interpol_im = 2.5 *  cat(3, gray_interpol_r, gray_interpol_g, gray_interpol_b);

white_interpol_r = interp2(white_bal_r);
white_interpol_g = interp2(white_bal_g);
white_interpol_b = interp2(white_bal_b);
white_interpol_im = 3.15 * cat(3, white_interpol_r, white_interpol_g, white_interpol_b);
```

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
