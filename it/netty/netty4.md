# netty启动分析

io.netty.eventLoopThreads设定bossGroup线程数，默认是CPU核数*2。

遍历ByteBuf

        for (int i = 0; i < buf.readableBytes(); i++) {
            System.out.print(buf.getByte(i)+" ");
        }

buf的readIndex不会改变。

        while (buf.isReadable()){
            System.out.print(buf.readByte()+" ");
        }

buf的readIndex会改变。
