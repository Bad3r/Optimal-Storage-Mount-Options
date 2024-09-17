- ## Recommended mount flags (Ext4)
	- Root partition
	  ```
	  defaults,noatime,barrier=1,errors=remount-ro,commit=60
	  ```
	- External NVME SSDs
	  ```
	  defaults,noatime,barrier=1,errors=remount-ro,commit=60
	  ```
	- External HDD (Focused on Data Integrity and Longevity)
	  ```
	  defaults,noatime,barrier=1,data=orddefaults,noatime,barrier=1,data=ordered,errors=remount-ro,commit=300
	- After changing filesystem options, update settings in all initramfs images:
		- Arch Linux 
	  ```
		  $ sudo update-initramfs -u -k all  
	  ```
		- Debian
	  ```
		  $ sudo update-initramfs -u -k all  
	  ```
- ## Options Explanation
	- **`defaults`**: This is a shorthand for using the default mount options, which include options like `rw` (read/write), `suid`, `dev`, `exec`, `auto`, `nouser`, and `async`. These are generally safe to use and provide standard functionality.
	- **`noatime`** is used universally to prevent unnecessary access time updates, which improves performance and reduces wear on SSDs and HDDs alike.
	- **`barrier=1`** is critical for data integrity, ensuring all data is safely written to disk.
	- **`errors=remount-ro`**: Remounts the filesystem as read-only in case of errors.
	- **`commit=60/120/300**` are used to balance data integrity and performance. A shorter interval (`commit=60`) is chosen for the root partition, where stability is crucial, while a longer interval is used for external drives to reduce the frequency of writes. Note that the default value is `5`
	- **`data=ordered`** is applied only to HDDs, ensuring the correct order of writes and improving data consistency in the event of a crash.
	-
- ## FAQ
	- Why is Ext4 used over other filesystems?
		- Since Ext4 is the most mature filesystem its recommended unless you have a reason not to use it.
	- Why is the `discard=async` option not used
		- Since `fstrim.timer` is enabled via systemctl. It is not recommended to use the discard option with it to avoid conflicts or redundant TRIM operations.
			- **`fstrim.timer`** vs **`discard=async`**
				- **`fstrim.timer`** runs TRIM periodically (typically once a week), which is better suited for avoiding performance hits during file operations. This is generally the preferred method for most SSDs because it defers TRIM operations to a scheduled time, minimizing impact on regular usage.
				- **`discard=async`** performs TRIM in real time but asynchronously, meaning it will not block file operations. It is more efficient than the older synchronous `discard` option, but it can still introduce a performance overhead when working with very large numbers of file deletions.
	- **Compression**
		- The compression option is not used with Ext4 since it does not support it unlike Btrfs and ZFS
		- Btrfs specific options
			- With Btrfs its recommended to add the option `compress-force=zstd:3`. Compression level 3 is the default so it can be committed. Consider compression level `1` for better performance.
				- unlike the `compress` option `compress-force` applies compression to newly written data disregarding Btrfs checks if the data is compressible. Instead zstd does a check internally that's more efficient than the one Btrfs does.
			- add `space_cache=v2` and `ssd`
- ## General recommendations
	- Enable `fstrim.timer`
	  ```
	  $ systemctl enable --now fstrim.service  
	  ```
		- Note that veracrypt blocks trim operations for security reasons.
	- Use `smartctl -a` to check for issues
	  ```
	  $ sudo smartctl -a /dev/<dev>  
	  ```
	- Use RAMDISK to reduce write frequency
	  ```
	  $ sudo cp /usr/share/systemd/tmp.mount /etc/systemd/system/  
	  $ sudo systemctl enable --now tmp.mount  
	  ```
	- use RAMDISK for `/tmp` in  `/etc/fstab`
	  ```
	  tmpfs   /tmp    tmpfs   defaults,noatime,mode=1777,size=8G,uid=0,gid=0   0  0  
	  ```  
- ## References
	- [SSDOptimization — Debian Wiki](https://wiki.debian.org/SSDOptimization)
	- [Solid State Drives  — Arch Wiki](https://wiki.archlinux.org/index.php/Solid_State_Drives)
	- [SSD Partitioning, Partition Alignment, Optimal Configuration Settings and Performance Testing — siduction.org](https://siduction.org/2012/01/ssd-partitioning-partition-alignment-optimal-configuration-settings-and-performance-testing/)
	-
