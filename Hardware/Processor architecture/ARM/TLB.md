# TLB

从虚拟地址到物理地址的转换过程可知：使用一级页表进行转换时，每次读写数据需要访问两次内存，第一次访问一级页表获得物理地址，第二次才是真正的读写数据；使用二级页表时，每次读写需要访问3次内存，访问两次页表（一级页表和二级页表）获得物理地址，第三次才是真正的读写数据。

上述的转换过程大大降低了CPU的性能，有没有什么办法改进呢？程序执行过程中，所用到的指令、数据的地址往往集中在一个很小的范围内，其中地址、数据经常多次使用，这称为程序访问的局部性。由此，通过使用一个高速、容量相对较小的存储器来存储近期用到的页表条目（段、大页、小页、极小页描述符），以避免每次地址转换时都到主内存中去查找，这样可以大幅度的提高性能。这个存储器用来帮助快速的进行地址转换，称为“转译查找缓存”（Translation Lookaside Buffers， TLB）。

当CPU发出一个虚拟地址时，MMU首先访问TLB。如果TLB中含有能转换这个虚拟地址的描述符，则直接利用词描述符进行地址转换和权限检查；否则，MMU访问页表找到描述符后再进行地址转换和权限检查，并将这个描述符填入TLB中（如果TLB已经填满，则使用round-robin算法替换），下次再使用这个虚拟地址的时候就可以直接从TLB中找到了。