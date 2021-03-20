### 계산영상시스템 Homework #1 : Demosaic

## Implementation a basic image processing pipeline

# Initials
Image read, convert to 2D-double array

```
clc; clear all;

img_cr2 = imread('G:\수업\수강\[2-1]계산영상시스템\hw\1\assign1\data\banana_slug.cr2');
img_tiff = imread('G:\수업\수강\[2-1]계산영상시스템\hw\1\assign1\data\banana_slug.tiff');

% image size 
[H, W] = size(img_tiff);

class(img_tiff)
double_img = double(img_tiff);
```
# Linearization
Linear tranformation for img and clipping
```
% Linearization into range [0,1] using linear transformation
norm_img = (double_img/(15000 - 2047)) - (2047 / (15000-2047));
% clipping
norm_img(norm_img<0) = 0;
norm_img(norm_img>1) = 1;
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/HaengInThere/IIT6028_2021_1/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
