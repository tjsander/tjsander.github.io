---
layout: post
title: "Using the iOS Camera with Swift"
image: https://farm7.staticflickr.com/6160/6205964186_1140d1048f_b_d.jpg
permalink: swift-camera
tags: code ios featured
---

One of the major barriers to overcome in programming for iOS and other Apple platforms is learning to work with Objective-C, the main language in all Apple operating systems since the release of OSX in 2001. With the recent introduction of the Swift language, this is no longer necessarily the case. In the announcement Apple described Swift as "Objective-C without the C", which may be a relief to many previously turned off by Objective-C's unique syntax (or C pointers).

While the introduction of the Swift programming language makes native iOS programming more accessible to those of us with little Objective-C experience, guides on Swift programming for iOS are still relatively sparse. Much of the documentation and an overwhelming majority of community resources are still written in Objective-C. It's relatively easy to find guides written on how to perform simple tasks using Objective-C, like [this guide on how to make a simple camera app in Objective-C,](http://www.appcoda.com/ios-programming-camera-iphone-app/) but the Swift equivalent is somewhat lacking.

This guide will demonstrate how to make a similar simple camera app in Swift including simple alerts and some core principles involved in iOS programming.

## Getting Started

A comprehensive guide to the Swift programming language can be found [here](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/index.html), or you can use this handy [cheat sheet.](http://cdn1.raywenderlich.com/wp-content/uploads/2014/06/RW-Swift-Cheatsheet-0_5.pdf) The syntax should be familiar to anyone who has experience with an object-oriented programming language.

Of course, you'll need to be working in OSX with the latest version of Xcode. If you wish to test apps on hardware, you'll need an [iOS Developer License.](https://developer.apple.com/programs/ios/) The iOS emulator cannot test the camera function.

Apple has also published a very nice guide to [getting a simple photo filtering app up and running in Swift.](https://developer.apple.com/swift/blog/?id=16) The application you will have after following the video is a good starting point.

## ViewController as a Delegate

First, extend the ViewController class inside of ViewController.swift to include UINavigationControllerDelegate and UIImagePickerControllerDelegate:

{% highlight Swift %}
class ViewController: UIViewController, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
{% endhighlight %}

Specifying a delegate allows your main ViewController window to call the UIImagePicker and execute actions specific to your app. In this case, it will allow us to retrieve images taken by the camera to replace the image in the default image view. For more on the concept of delegation, see [the documentation.](https://developer.apple.com/library/ios/documentation/general/conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html)

## Call UIImagePickerController

Just like in the demo for the photo filtering app, make a new button and control+click and drag to the ViewController to create a new Action.

![](/assets/images/ctrlclick.png "AND DRAG!")


{% highlight Swift %}
@IBAction func takePicture(sender: AnyObject) {
    if (UIImagePickerController.isSourceTypeAvailable(UIImagePickerControllerSourceType.Camera)){
        var picker = UIImagePickerController()
        picker.delegate = self
        picker.sourceType = UIImagePickerControllerSourceType.Camera
        picker.mediaTypes = [kUTTypeImage]
        picker.allowsEditing = true
        self.presentViewController(picker, animated: true, completion: nil)
    }
    else{
        NSLog("No Camera.")
    }
}
{% endhighlight %}

The code first checks to make sure that the camera is available. If it is not, the function ends. Otherwise, it instantiates a new UIImagePickerController, delegates to the ViewController (self). In this example, allowsEditing is set to true, so that the picker will return both the original image as well as a square selection. We then set the sourceType to Camera and the mediaTypes to [kUTTypeImage] and call self to bring up the picker.

## BONUS: Debug Statements and Alerts

The NSLog statement in the previous code snippet is a good way to get debug information while testing your app. It may be useful to include some sort of alert to the user that the feature requires a camera, in case the user is using a device without a camera or the user has not allowed the app to access the camera. If your app's functionality is wholly dependend upon devices with cameras, it may be a good idea to limit its App Store availability. Be sure to addAction to the alert in order to allow the user to dismiss it, as this is not included by default.

{% highlight Swift %}
NSLog("No Camera.")
var alert = UIAlertController(title: "No camera", message: "Please allow this app the use of your camera in settings or buy a device that has a camera.", preferredStyle: UIAlertControllerStyle.Alert)
alert.addAction(UIAlertAction(title: "Dismiss", style: UIAlertActionStyle.Default, handler: nil))
self.presentViewController(alert, animated: true, completion: nil)
{% endhighlight %}

## Handle the Results

[UIImagePickerControllerDelegate](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImagePickerControllerDelegate_Protocol/index.html) objects have two functions to handle output from the picker (in this case, the camera). One where the camera returns some media, and the other where the user cancels. Copy the following inside the ViewController class in ViewController.swift
{% highlight Swift %}
func imagePickerController(picker: UIImagePickerController, didFinishPickingMediaWithInfo info: NSDictionary!) {
    NSLog("Received image from camera")
    let mediaType = info[UIImagePickerControllerMediaType] as String
    var originalImage:UIImage?, editedImage:UIImage?, imageToSave:UIImage?
    let compResult:CFComparisonResult = CFStringCompare(mediaType as NSString!, kUTTypeImage, CFStringCompareFlags.CompareCaseInsensitive)
    if ( compResult == CFComparisonResult.CompareEqualTo ) {
        
        editedImage = info[UIImagePickerControllerEditedImage] as UIImage?
        originalImage = info[UIImagePickerControllerOriginalImage] as UIImage?
        
        if ( editedImage != nil ) {
            imageToSave = editedImage
        } else {
            imageToSave = originalImage
        }
        imgView.image = imageToSave
        imgView.reloadInputViews()
    }
    picker.dismissViewControllerAnimated(true, completion: nil)
}
{% endhighlight %}

The above function activates when the picker returns with media. If you set the picker to allow editing, the picker will return with both the original as well as the edited image. It then does additional error checking to make sure the mediaType is correct before retreiving the image data. The code checks if the edited image is nil and defaults to the original image if it is. Otherwise, it sets an imageView object (control-dragged to your view as an outlet, as in the Apple demo) to the captured image. At this point you can have your code do anything you want with the image data, including save it to the camera roll with UIImageWriteToSavedPhotosAlbum, saved to the filesystem of your app, sent to the cloud, and so on.

{% highlight Swift %}
func imagePickerControllerDidCancel(picker: UIImagePickerController) {
	picker.dismissViewControllerAnimated(true, completion: nil)
}
{% endhighlight %}

This function handles cases where the user decides to cancel the camera, and from my tests it seems to be entirely optional. The ViewController will dismiss itself upon Cancel. However it seems to be a best practice to include a handler for this case as well as any additional instructions.

At this point, my app looks like this:
![](/assets/images/appfilter.png)

## Conclusion

The concept illustrated above is useful in any situation where you want to retrieve an image from the user's camera. A further extension of this project would be to allow the user to select either a picture from the Photo Gallery or the Camera.

*Hint: use UIImagePickerControllerSourceTypePhotoLibrary (Seriously.)*

In the case of applications which may require camera access from many different views, it may be more efficient to impliment the above code as its own class. The image below represents my current efforts to access the camera that way:

![](https://farm2.staticflickr.com/1001/1357938629_5c217662d3_z_d.jpg "https://www.flickr.com/photos/rickyromero/1357938629")

*Header image by [gabrielap93](https://www.flickr.com/photos/gabrielap93/6205964186)*

Code in this how-to modified from Harold Dost III's [CameraVC class](http://blog.raastech.com/2014/09/using-camera-in-ios-8-with-swift.html)

For more information on how to take direct control of the iOS camera in Swift using newly-available APIs in iOS8, see [this two-part guide.](http://jamesonquave.com/blog/taking-control-of-the-iphone-camera-in-ios-8-with-swift-part-1/)

