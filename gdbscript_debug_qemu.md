编译脚本

./configure --prefix=/usr --target-list=x86_64-softmmu --enable-kvm --enable-debug  --enable-debug-info --enable-modules --enable-vnc --enable-trace-backends=log --disable-werror --disable-strip

	set confirm off
	set pagination off
	set disassembly-flavor intel
	handle SIGALRM nostop nopass noprint
	disp /i $pc

	define p
	si
	end

	define o
	ni
	end

	define l
	x/16i $arg0
	end

	define ln
	x/16i
	end

	define d
	x/16xg $arg0
	end

	define dn
	x/16xg
	end

	define ds
	x/s $arg0
	end

	define dr
	print/x $$arg0
	end

	define pv
	print/x {int}$arg0
	end

	define pc
	x/i $pc
	end
