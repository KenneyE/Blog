---
layout: post

title: Design Patterns in Ruby - The Abstract Factory
subtitle: "An Example of the Abstract Factory Pattern Implemented in Ruby"
cover_image: blog-cover.jpg

excerpt: "This is a basic not-quite-functioning example of the Abstract Factory Pattern from the book 'Design Patterns: Elements of Reusable Object-Oriented Software' implemented in Ruby. It uses the same Maze Game example for demonstration as the book in order to make the book's discussion as relevant as possible."

author:
  name: Eric Kenney
  bio: Web Developer
  image: logo.png
---

##The Pattern
There are 2 important parts of Abstract Factory pattern. The first is **The Factory** itself. This separates the client from possibly many sub-types of factories. The second is **The Product**, which also needs to be abstract so that no matter which sub-type of factory is being used, the end products need to all look the same as far as the client is concerned. So when a *Map Factory* can create a *Treasure Map*, a *Star Map* or a *Trail Map*, it still needs to look like just a *Map* to the client.

### The Factory
The Abstract Factory pattern is used to isolate the client from the possibly large number and variety of products in an application. We'll use the example of a video player. The abstract *VideoPlayerFactory* class is aware of each sub-type of video player factory and each factory method that those sub-types contain. These factory methods could include `make_mute_button`, `make_progress_bar`, and `make_volume_slider`. Every sub-type of factory would implement each of these functions differently, and the abstract factory doesn't care. The sub-types of video player factories could be *AdVideoPlayerFactory*, *ChristmasThemeVideoPlayerFactory* or *StreamingVideoPlayerFactory*.

Even though each factory sub-type inherits from the abstract factory, each sub-type should have different behaviors and appearances for things like the mute button and progress bar. For example, the streaming video player's progress bar might show that it's always at the end of the video, but allow users to seek to previous parts of the stream. So each sub-type would implement the factory methods differently. The client doesn't need to know this though. All they need to be able to do is tell the *VideoPlayerFactory* that they would like a video player to be made, and tell it what kind it should be. 

### The Products (mute button/scroll bar/etc.)

With this pattern, not only should the client not need to worry about the different types of factories being used; he/she should also not care about what the results products look like. In order to achieve this, the products should also inherit from abstract products that the client knows about. For example, the *StreamingVideoPlayerFactory* will probably create a *StreamingProgressBar*. This should inherit from the *ProgressBar* class, just like each of the other types of progress bars. The client only knows about this abstract class. This way, the client should not need to make any changes to their program when they decide to use a *ChristmasThemeVideoPlayer* instead. All they need to do is tell the abstract factory to change which sub-type it creates. In the end, the mute button, scroll bar, and volume slider all function the same way as far as the client is concerned, regardles of video player type.

##Abstract Factory in Ruby