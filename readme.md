This repo is transfer to [utilities-library](https://github.com/wangmengHB/utilities-library) for easily management, which is a multiple npm packages repo managed by lerna. The npm package will still be maintained continously. 

This repo will be deleted in one month later!!!.


# gl-image
gl-image is a util library based on webgl for image filter functions.   
This project is inspired by [@evanw](https://github.com/evanw/glfx.js).

Here is the main difference between glfx.js and gl-image:    
glfx.js just support one shader program at one time.    
But in many use case, we need to do multiple filters to one image, and every operation can be redoable.
So gl-image do something more to support multiple filters (multiple shader programs inside at once).

And it is really simple for use.

![demo1](https://raw.githubusercontent.com/wangmengHB/gl-image/master/demo/demo.jpg)

# install
```bash
npm install --S gl-image
```
```ts
import GLImage from 'gl-image';
const glImage = new GLImage();

glImage.loadImageSrc(src).then(() => {
  // do filter for image
  glImage.applyFilter('brightness', 0.3);
  glImage.applyFilter('hue', -0.3);
});
```
### filters supported 
| filter type name   | default value     | range       |
| :---------         | :-------          | :---------- |
| brightness         | 0                 | [-1, 1]     | 
| contrast          | 0                 | [-1, 1]     | 
| hue                | 0                 | [-1, 1]     | 
| saturation         | 0                 | [-1, 1]     | 
| sepia_amount       | 0                 | [0, 1]      | 
| vibrance_amount    | 0                 | [-1, 1]     | 
| vignette_amount    | 0                 | [0, 1]      | 
| vignette_size      | 0                 | [0, 1]      | 
| noise_amount      | 0                 | [0, 1]      | 
| pixelate_block_size | 0               | [0, 20]     |


default value in the above table means no filter effect.

###### note: 
1. please make sure the image source url is cross-origin supported, otherwise browser will block reading pixels from the image. gl-image will failed to do anything.
2. please make sure your browser supports webgl. 
3. this npm package cannot run in nodejs environment.  


# API
```ts
import GLImage from 'gl-image';
const glImage = new GLImage();
```
In most cases, one GLImage instance is enough for usage, you don't need create a new one to handle another image.   
## 1. load source image from image/canvas/url 
*  `async loadFromElement(source: HTMLImageElement | HTMLCanvasElement)`
*  `async loadImageSrc(url: string)`  

## 2. do filter action
* `applyFilter(filterName, filterValue)`    
* `applyFilters({name1: value1, ...})`      
Please refer to the above table to find the available filter name and valid value range.  


## 3. get the output

For performance consideration, it is using `preserveDrawingBuffer: false` mode inside. So that you would lose drawing data immediately in another event loop. So it is necessary to have another 2D canvas inside to store the drawing result.  
The output is stored in another 2D canvas for outside usage. 

* `getCanvas()`: get the result canvas element which store the result image;      
* `getImageData()`: get the ImageData from result canvas element;     
* `toDataULR(type?, quality?)`: get the data url from result canvas element;    

You can use the result canvas in any way you like. 

# usage

## use case 1. simple use
```ts
import GLImage from 'gl-image';

let glImage = new GLImage();

glImage.loadImageSrc(src).then(() => {

  // do single filter for image
  glImage.applyFilter('brightness', 0.3);
  glImage.applyFilter('hue', -0.3);

  // do batch filter for image
  glImage.applyFilters({
    'brightness': 0.3,
    'saturation': -0.7
  });

  // you can the base64 output data.
  const base64 = glImage.getDataURL();

});
```
## use case 2: batch process images
```ts
import GLImage from 'gl-image';

const glImage = new GLImage();

const imageSrcList = ['xxx.png', 'yyy.png', ...];

async function processSingle(imageSrc) {
  await glImage.loadImageSrc(imageSrc);
  glImage.applyFilters({
    'brightness': 0.3,
    'saturation': -0.7
  });
  return glImage.getCanvas();
}

async function batchProcess(imageSrcList) {
  const result = [];
  for (let i = 0; i < imageSrcList.length; i++) {
    const res = await processSingle(imageSrcList[i]);
    result.push(res);
  }
  return result;
}

batchProcess(imageSrcList).then((result) => {
  console.log(result);
})

```

# Notice:
`getDataURL()` is retired.       
Please call `getCanvas()` instead.
You can get the result 2D canvas element which store the result, then you can do anything on this canvas in any way you like.

`fromLoadedImage()` is retired. please use `loadFromElement` instead. 


