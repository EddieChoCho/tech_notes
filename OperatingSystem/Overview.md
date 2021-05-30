# Historical Prospective & Overview

* Mainframe: Time-sharing System(Multi-tasking System)
	* OS tasks
		* `Virtual memory`(ch10)
			* Use disk as memory
			* Jobs swap in and out of memory to obtain reasonable response time
		* `File system` and `disk management`(ch11,12)
		* `Process synchronization` and `deadlock`(ch7,8): support concurrent execution of programs

* Clustered System
	* Cluster computers share storage and are closely linked via a local area network(LAN) ir a faster interconnect, such as InfiniBand(up to 300Gb/s).

* Real-TIme Operating System(ch19)
	* Well-defined fixed-time constraints. "Real-time" doesn't mean speed, but keeping deadlines
	* Guaranteed response and reaction times
	* Often used as a control device in a dedicated applications
		* Scientific experiments, medical imaging system, industrial control systems, etc
	* Real-time requirement: `hard` or `soft`

* Soft v.s. Hard Real-Time
	* Soft real-time requirements
		* Missing the deadline is unwanted, but is not immediately critical
		* A critical real-time task gets `priority` over other tasks, and retains that priority until it completes.
		* Examples: multimedia streaming
	* Hard real-time requirements
		* Missing the deadline results in `a fundamental failure`.
		* `Secondary storage limited` or `absent`, data stored in short term memory, or read-only memory(ROM)

* Issues of Multimedia System(Ch20)
	* Timing constraints: 24~30 frames per second
	* On-demand/live streaming: media file is only played but not stored
	* Compression: due to the size and rate of multimedia systems

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/playlist?list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX)