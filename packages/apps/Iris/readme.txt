This demo app show that how to get the pureraw data from sensor.

The pureraw data is from the PreviewCallback function onPreviewFrame.

The preview callback size maybe is not same as sensor size.

For example the sensor size on demo phone is 1280*960.

But the previewcallback size is large than 1280*960.
The previewcallback data is yuv format. The pull raw sensor image is stored in from of y plane of previewcallback buffer.

The data of later y plane data is not used.

Note:
Please change the the sensor size on Iris demo app.

