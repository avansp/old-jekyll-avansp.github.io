---
layout: post
title:  "Image Creations with ITK, OpenCV and Qt"
date:   2014-12-16
comments: true
---

Three C++ libraries I'm currently using to handle image data: ITK image, OpenCV's Mat and Qt's QImage. They are different and are not compatible one to each other. Why do I need those? That's because each comes with its own strength. ITK is a huge library of hundreds of medical imaging filters that were the products of well-known research groups. OpenCV comes from computer vision communities, who are very good in robotic, video processing and general image processing. Qt is an open-source library for user interface and I need Qt to bundle my application.

The problem now is that I need to convert images between these three. Before that, I need to make sure that image pixel iterators run correctly.

## `itk::Image`

ITK is a heavily templated library. To create an image object, you must define what type the pixels are and the image dimension. Then you define the image size and image region. Note that ITK uses region as a distinct entity than the size, because an ITK filter can be threaded; meaning that you can chunk a huge image into disjoint regions to be processed in different cores. The `LargestPossibleRegion` is the whole image size.

To create a two-dimensional `itk::Image` object with `unsigned short` (16-bit) pixel type, it's a mouthful piece of codes:
{% highlight c++ %}
// define the 2D image with unsigned integer pixel type
typedef itk::Image< unsigned short, 2 > ImageType;

// define the origin: usually at (0,0)
ImageType::IndexType start;
start[0] = start[1] = 0;

// define the image size: e.g. 128 x 512 (just for a test)
ImageType::SizeType size;
size[0] = 512;  // width
size[1] = 128;  // height

// define the largest image region
ImageType::RegionType region;
region.SetSize(size);
region.SetIndex(start);

// create image: don't forget to allocate
ImageType::Pointer theImage = ImageType::New();
theImage->SetRegions(region);
theImage->Allocate();
{% endhighlight %}

Traversal can be done using STL-type iterator:
{% highlight c++ %}
// STL-type iterator to the whole image
itk::ImageRegionIterator<ImageType> it( theImage, theImage->GetLargestPossibleRegion() );
unsigned short pxValue = 0;
for( it.GoToBegin(); !it.IsAtEnd(); ++it ) it.Set( pxValue++ );
{% endhighlight %}

or using indexing operator. However, **please note** that ITK uses `[width][height]` dimension, which means that indexing is taken in the form of `[col][row]` or `[x][y]`:
{% highlight c++ %}
// Indexing style should be carefully taken !!
unsigned short pxValue = 0;
ImageType::IndexType idx;
ImageType::RegionType allRegions = theImage->GetLargestPossibleRegion();
for( itk::SizeValueType row=0; row<allRegions.GetSize(1); ++row ) {
  idx[1] = row;  // NOTE: row is the second index of the IndexType
  for( itk::SizeValueType col=0; col<allRegions.GetSize(0); ++col )
  {
    idx[0] = col; // NOTE: col is the first index of the IndexType
    (*theImage)[idx] = pxValue++; // or: theImage->SetPixel(idx,pxValue++);
  }
}
{% endhighlight %}

Both snippets will give the following image:
![itk::Image ramp image]({{ base.url }}/images/itkImage-test-512x128.png)
Notice how ```itk::Image``` runs from the top-left as the origin and the iterator sweeps through columns (left to right) from the top to the bottom rows.

## `cv::Mat`

The OpenCV libray was not designed from the bottom up. Instead, it was developed through current needs of fast computing in computer vision. As such, there are mixed of C and C++ codes and also between templated and non-templated classes. The reason is backward compatibility, but it costs the price of consistency. The ```cv::Mat``` image data class was designed for easy indexing access; similar to Matlab's matrix data. The default `cv::Mat` is an image with integer pixel type, which is usually enough for computer vision problem. To use different types, OpenCV provides a templated `cv::Mat_` wrapper which should be used very very carefully.

The same way to create 16-bit pixel type with `cv::Mat`:
{% highlight c++ %}
// Create image with nrows=128, ncols=512, CV_8UC1=16 bit unsigned channels=1
cv::Mat theImage( 128, 512, CV_16UC1 );
{% endhighlight %}
That's it. Very easy.

Unfortunately, we can only iterate through all pixels (the STL way) by using a templated `cv::MatIterator_` class:
{% highlight c++ %}
// STL-iterator for cv::Mat
unsigned short pxValue = 0;
cv::MatIterator_<int> it;
for( it=theImage.begin<int>(); it != theImage.end<int>(); ++it ) *it = pxValue++;
{% endhighlight %}

The equivalent pixel indexing iteration is straightforward. OpenCV uses `[row][col]` or `[y]][x]` indexing:
{% highlight c++ %}
// Pixel indexing
unsigned short pxValue = 0;
for( int row=0; row<theImage.rows; ++row )
  for( int col=0; col<theImage.cols; ++col )
    theImage.at<unsigned short>(row,col) = pxValue++;
{% endhighlight %}

The image is:
![cv::Mat ramp image]({{ base.url }}/images/cvMat-test-512x128.png)
It's equal with `itk::Image`.

## `QImage`

`QImage` was designed by Qt as an IO image before visualizing it on a Qt widget. During visualization, a `QImage` object is converted into `QPixmap` or `QBitmap` object. There is no direct pixel access in a `QPixmap` or `QBitmap` object, because they are paint devices and setting pixel values are performed through painting. Usually, `QImage` is created through image reading, but we can manually create `QImage` object.

Creating 16-bit `QImage` is a bit tricky, because there is no monochrome format. `QImage` only provides a range of RGB formats.
{% highlight c++ %}
// Create QImage: width=512, height=128, format RGB888 means RGB format 8-8-8
QImage theImage(512, 128, QImage::Format_RGB888);
{% endhighlight %}

Traversing using an STL-iterator is not possible in Qt, but we can perform a naive iteration as long as the pointer is valid. The problem is that Qt only provides a pointer to `unsigned char`. So it leaves user to correctly assign values there.
{% highlight c++ %}
// Pixel traversing
unsigned char *buffer = theImage.bits();
unsigned short pxValue = 0;
for( int i=0; i<(theImage.width()*theImage.height()); ++i )
{
  unsigned char pxCharValue = (unsigned char) (pxValue / 256);

  // we have to set the same value three times because of RGB components
  *buffer++ = pxCharValue;
  *buffer++ = pxCharValue;
  *buffer++ = pxCharValue;

  ++pxValue;
}
{% endhighlight %}

The pixel indexing style is given below (note how `col,row` is given to the `setPixel` method):
{% highlight c++ %}
// Pixel indexing
unsigned short pxValue = 0;
for( int row=0; row<theImage.height(); ++row )
for( int col=0; col<theImage.width(); ++col )
{
  unsigned char pxCharValue = (unsigned char) (pxValue / 256);
  theImage.setPixel(col,row,qRgb(pxCharValue,pxCharValue,pxCharValue));
  pxValue++;
}
{% endhighlight %}

The image is:
![cv::Mat ramp image]({{ base.url }}/images/qImage-test-512x128.png)
which is the same with the previous two images.

## In conclusion

> Pointer-based iterations from ITK, OpenCV and Qt images work similarly: row by row (top to bottom) and for each row it runs from left to right. ITK and OpenCV are straightforward to use, but Qt does not provide STL.

One important note is the difference in the matrix dimension. ITK image dimension: `[width][height]`. OpenCV image dimension: `[rows][columns]`. Qt does not provide direct indexing, but setting is given by `setPixel(col,row)`.
