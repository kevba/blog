---
title: "Go Garbage and Files"
date: 2018-05-18T21:11:33+02:00
draft: false
---

### Some context

A while ago I wrote a small [Go library](https://github.com/AdvancedClimateSystems/io/tree/master/gpio/acme/g25) to work with the GPIO pins on an [Aria G25 chip](https://www.acmesystems.it/aria). This library can be used to read and write those pins. It also offers a function to set a callback on an edge (Rising, Falling or both). This means that when a pin changes its state, the callback got executed.

The main part of the event handling is the Watcher. The watchers uses epoll to watch multiple files for events. [epoll](http://man7.org/linux/man-pages/man2/syscall.2.html) is an API of system calls to monitor a file descriptor for events. This API has been implemented in the [Go standard library](https://golang.org/pkg/syscall/#EpollCreate). The following code snippet demonstrates how I used the epoll API.

```golang
func (w *watch) AddEvent(fpntr int, callback func()) error {
	var event syscall.EpollEvent
	event.Events = syscall.EPOLLIN | (syscall.EPOLLET & 0xffffffff)
	event.Fd = int32(fpntr)

	// An application that employs the EPOLLET flag should use nonblocking
	// file descriptors to avoid having a blocking read or write starve a
	// task that is handling multiple file descriptors.
	//  - http://man7.org/linux/man-pages/man7/epoll.7.html
	if err := w.sysH.SetNonblock(fpntr, true); err != nil {
		return err
	}

	if err := w.sysH.EpollCtl(w.fd, syscall.EPOLL_CTL_ADD, fpntr, &event); err != nil {
		return err
	}
	w.addCallback(fpntr, callback)
	return nil
}
```
The calls to EpollCreate1(), EpollCtl() and EpollSetNonblock() initialize an epoll instance. The inner workings of these functions are not important here. Then EpollWait is called. This function blocks, until it detects an event that is configured using Epoll.Events on the file descriptor denoted by EpollEvents.Fd. The snippet above uses the file descriptor of the value file of the GPIO pin. Note that this file descriptor is just an int. The file descriptor is also used as the key for the callback function.

### The Bug

This seemed to work very well for cases where there weren’t many triggers, such as a button. However an issue arose when using the pin as a pulse counter. At first the counter happily counted all the incoming pulses. This worked well for the first couple of days, but then it would suddenly stop counting. No error message was given, it just hanged. After restarting the software it would work again like nothing had ever gone wrong.

Since I needed the pulse counter to work, an old python script was used to do the counting. This Python script did not have the same issues as the Go one, so there had to be a bug must in the Go code.

### Reproducing

Since the bug only occured after a few days naturally, a quicker way to reproduce it was needed. First I  needed something that generates pulses. This was achieved by creating a script that quickly toggled a GPIO pin in output mode.

Next I connected the GPIO pin used for the pulses to another GPIO pin. This pin was set up as an input with an callback on a rising edge.

The bug could now be reproduced by running this script for about 5 minutes.

### Debugging

My approach to debugging mostly consists of starting to top and working my way down. So the first thing to check was the callback. The [handleEvent][handleEvent] function calls a given function when an edge trigger happens. I wrote a very simple callback function which would log the amount of times it was executed. As expected, after a while the callback as no longer executed. This meant the callback itself was not the problem.

Next I checked if handleEvent itself was still executed when the bug happened. Just as with the callback itself, handleEvent was no longer executed.

This brought me to the [Watch][Watch] function. The Watch function is an endless loop that uses epoll to wait for events. If an event occurred on a GPIO pin, it would executed the callback for that pin by using its file descriptor. Interestingly after a while no new events happened at all.

This meant that either the epoll setup was wrong or there is something wrong with the files themselves. Since epoll did not give any errors whatsoever I decided to check the files themselves.

This is where things got interesting. Because I didn’t get any new events from the watched files, I decided to read them in the Watch loop. Because I only had the file descriptor, I used [syscall.Read](https://golang.org/pkg/syscall/#Read) to read the file. As expected there was no data to be read from the descriptor, but something interesting happened when the bug happened.

![An error message!](/images/go-garbage-and-files/log-message.png)

Huh, an error message. Somehow the file descriptor from the watched file has become bad.

Did this mean the file was closed somehow? I added a defer call to [SetEdge][SetEdge] which would close the file after 10 seconds of opening it. I did this in SetEdge because it opens the file and passes the file descriptor to [AddEvent][AddEvent]. As expected, the same bug occured, exactly when the file got closed.

Then it hit me: what happens when the file got garbage collected? The file never got closed, but I did not keep a reference to it anywhere, since a file descriptor is just an integer. So maybe when the file object itself got garbage collected, the file got closed and the file descriptor becomes invalid.

### The fix

This is easy to test, just pass a pointer to the file to AddEvent and store it in the callback. If this truly was the problem, the bug should be gone.

So I ran the script. And waited. Then waited some more. After 10 minutes and 30,000 callbacks the script was still functioning.

Great, now I know how to fix it, but I'm still not sure why this happens. So i decided to dive into the Go source code, specifically [os.file](https://golang.org/src/os/file_unix.go).
After some searching I noticed a function called [SetFinalizer](https://golang.org/src/os/file_unix.go?#L132). This function gets called in newFile, which gets called when a new file is opened. A function given as an argument to Finalizer gets called when the object given as the first arguemnt gets garbage collected. In the case of our file, the file gets closed.
This perfectly explains why the file descripter became invalid. The file simply got closed!   

### Wrap up
This was probably one of the most interesting bugs I’ve encountered. It highlights just how much there is to know about a language and the things surrounding it, such as the garbage collector.

Finalizers are an interesting thing which I would have never known about, were it not for this bug.

Thanks to [OrangeTux](orangetux.nl) for helping me explain epoll!

[handleEvent]: https://github.com/AdvancedClimateSystems/io/blob/0694abfdd08ab4e334fc34f4c6a1581772208386/gpio/watch.go#L101
[Watch]: https://github.com/AdvancedClimateSystems/io/blob/0694abfdd08ab4e334fc34f4c6a1581772208386/gpio/watch.go#L64
[AddEvent]: https://github.com/AdvancedClimateSystems/io/blob/0694abfdd08ab4e334fc34f4c6a1581772208386/gpio/watch.go#L122
[SetEdge]: https://github.com/AdvancedClimateSystems/io/blob/0694abfdd08ab4e334fc34f4c6a1581772208386/gpio/gpio.go#L208
