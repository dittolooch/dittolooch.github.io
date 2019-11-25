---
layout: post
title: How to remove Main.Storyboard in xCode (iOS 13)
tags: [ios, swift, xcode]
---

![swift](https://user-images.githubusercontent.com/44837996/69522494-7de9a380-0f9c-11ea-8480-d432eff0f3c2.jpg)

> I had spent about 2 weeks on the app project when I realised that I made a big mistake: I started implementing everything on the Storyboard! 

As the project got bigger, the storyboard got extremely slow and my development efficiency plummetted. I would have said that whether to use storyboards in your iOS development project or not is a widely debated issue. But it really isn't. Unless you are building something that's very simple, storyboards are going to kill your efficiency, period. (and if you're really building something simple, guess what, there just might be an app for that already)

There are tons and tons of videos and articles on the pros and cons of using Storyboards. So I won't get into that topic here.

With the changes that came along iOS 13, a lot of the tutorials that teach you how to delete the storyboard and initiate your own root view controller are just obsolete. It took me about an hour to find this solution so I'd like the share with anyone who comes across this article.

So now you have your Swift project ready, you can follow the steps below to delete your story board and initiate your own root view controller!

## Delete the Storyboard

I don't think it needs an image for illustration. Just go ahead and right click on your Main.storyboard and delete it (move to trash)!

## Edit Deployment Info

![swift](https://user-images.githubusercontent.com/44837996/69523341-aa062400-0f9e-11ea-8f5b-c91f6a5856d5.png)

Go to your-project -> targets -> general -> deployment info -> Main Interface (as shown in the image above)
Remove the "Main" text. Simply leave it blank.

## Remove Main.storyboard reference in info.plist

![plist](https://user-images.githubusercontent.com/44837996/69523707-84c5e580-0f9f-11ea-8220-7bc52d3cd85e.png)
There is a Info.plist file within your project. There is a line referring to Main.storyboard buried deep in the text.
It is Application Scene Manifest -> Scene Configuration -> Application Session Role -> Item 0 (Default Configuration) -> Storyboard Name.
Click on the minus sign icon as shown in the image to remove it.

## Edit ViewController.swift

Go to the ViewController.swift file and change the background color of the view to red. It is for us to be able to check whether this view controller is correctly initiated by our code.

{% highlight swift %}

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        view.backgroundColor = .red
    }

}

{% endhighlight %}

## Edit SceneDelegate.swift

In SceneDelegate.swift file, find the ```scene(_ scene: UIScene, willConnectTo session:UISceneSession...)``` function and initiate your root view controller there.

{% highlight swift %}

func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    window = UIWindow(frame: windowScene.coordinateSpace.bounds)
    window?.windowScene = windowScene
    let rootVC = ViewController()
    window?.rootViewController = rootVC
    window?.makeKeyAndVisible()
}

{% endhighlight %}

## Build and Run

That's it! Build and run! Now you have programmtically initiated your bloody red view controller without storyboard!
![done](https://user-images.githubusercontent.com/44837996/69524552-403b4980-0fa1-11ea-86ec-9619d4c5d983.png)

Please give me some claps if you find this useful!

Happy coding.