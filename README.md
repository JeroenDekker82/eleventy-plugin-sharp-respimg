# eleventy-plugin-sharp-respimg
An Eleventy [paired shortcode](https://www.11ty.dev/docs/shortcodes/#paired-shortcodes) that performs build-time image transformations with [Sharp](https://sharp.pixelplumbing.com/) to resize large images into `.jpeg` and `.webp` formats with varying dimensions and generates `<picture>` tags for responsive images.

## Installation
In your Eleventy project, [install the plugin](https://www.npmjs.com/package/eleventy-plugin-sharp-respimg) from npm:
```
npm install eleventy-plugin-sharp-respimg
```
Then add it to your [Eleventy Config](https://www.11ty.dev/docs/config/) file:
```js
const respimg = require("eleventy-plugin-sharp-respimg");

module.exports = (eleventyConfig) => {
    eleventyConfig.addPlugin(respimg);
}
```

## What does it do?
It turns paired shortcodes like this:

```js
{% respimg 
    "car.jpg", 
    "Photo of a car", 
    "./images/",
    { small: 320, med: 640, large: 1024 },
    "(min-width: 450px) 33.3vw, 100vw",
    "my-image",
    "1024",
    "768"
%}{% endrespimg %}
```
into responsive image markup using `<picture>` tags like this:
```html
 <picture>
    <source 
        type="image/webp"
        srcSet="/images/car-large.webp 1024w,
                /images/car-med.webp 640w,
                /images/car-small.webp 320w"
        sizes="(min-width: 450px) 33.3vw, 100vw"
    >
    <img 
        srcSet="/images/car-large.jpg 1024w,
                /images/car-med.jpg 640w,
                /images/car-small.jpg 320w"
        sizes="(min-width: 450px) 33.3vw, 100vw"
        src="car-small.jpg"
        alt="Photo of a car"
        loading="lazy"
        class="my-image"
        width="1024"
        height="768"
    >
</picture>
```
- The images are responsive by using a `<picture>` element which contains zero or more `<source>` elements and one `<img>` element to offer alternative versions of an image for different display/device scenarios. 
- Using `srcset` and `sizes`, you can deliver [variable-resolution images](https://www.smashingmagazine.com/2014/05/responsive-images-done-right-guide-picture-srcset/), which respond to variable layout widths and screen densities.

### Using the paired shortcode more than once for the same image
If you have already used the utility to transform an image and you call `respimg` within your code again for the same file, it will only generate the responsive image markup using `<picture>` for that image and skip the image transformation as it checks the file system to make sure those resized images already exist.

## Transform mulitple images
The real power of using this paired shortcode is the ability to use data from [global data files](https://www.11ty.dev/docs/data-global/) or [front matter](https://www.11ty.dev/docs/data-frontmatter/) to transform multiple images at once.

If you have global JSON data stored in `data.json` which is an array of objects like this:

```json
[
    {
        "src": "car.jpg",
        "alt": "Photo of a car",
        "imgDir": "./images/",
        "widths": {
            "small": 320,
            "med": 640,
            "large": 1024
        },
        "sizes": "(min-width: 450px) 33.3vw, 100vw",
        "class": "my-image",
        "width": 1024,
        "height": 768
    },
    {
        "src": "flower.jpg",
        "alt": "Photo of a flower",
        "imgDir": "./images/",
        "widths": {
            "small": 400,
            "med": 600,
            "large": 1024
        },
        "sizes": "(min-width: 450px) 33.3vw, 100vw",
        "class": "my-image",
        "width": 1024,
        "height": 768
    }
]
```
you can use the paired shortcode to transform multiple images with varying dimensions into responsive image markup using a `for` loop like this:

```js
{% for image in data %}
    {% respimg 
        image.src, 
        image.alt, 
        image.imgDir,
        image.widths, 
        image.sizes,
        image.class,
        image.width,
        image.height
    %}{% endrespimg %}
{% endfor %}
```

## Paired shortcode options

| Parameter | Description |
| ------    | -------     |
| src       | The filename for an image. |
| alt       | A text description of the image. |
| image directory | The directory where the image file is located. |
| widths    | The desired image widths. Supports any three integer values. |
| sizes     | The `sizes` attribute which defines a set of media conditions. |
| class     | Class name for the fallback image.   |
| width     | The fallback image `width` attribute.  |
| height    | The fallback image `height` attribute. |

## Notes
- Use `./` when declaring the image directory parameter as Sharp expects this.
- Use `.addPassthroughCopy` to include the images directory in your `_site` output with `eleventyConfig.addPassthroughCopy("imageDirectory");`.
- The shortcode expects all the arguments to be defined, if you only need one or two widths rather than the required three values, I'm working on adding this support.
- The `<picture>` and `<img>` tags generated by the paired shortcode don't have any styling out of the box. But they can be manipulated with a bit of CSS to apply different `width` or `height` attributes.

Recommended fallback `<img>` boilerplate CSS:
```
.my-image {
    max-width: 100%;
    object-fit: cover;
    height: auto;
    margin: 0 auto;
}
```

## TODO
- [ ] If an image is smaller than the largest specified width value, decide if the image should be resized, as I don't want a small image transformed into larger dimensions than the intrinsic size.
- [ ] Allow the widths parameter to be an array of values rather than an object.
- [ ] Image quality is set at `quality: 85`, maybe allow `quality` to be an optional param if you need very high quality images.

## Other Responsive Image Plugins
- [eleventy-img](https://github.com/11ty/eleventy-img)
- [eleventy-respimg](https://github.com/eeeps/eleventy-respimg)