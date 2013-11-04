# 描述

=head1 DESCRIPTION

This document attempts to describe how to use the Perl API, as well as
to provide some info on the basic workings of the Perl core. It is far
from complete and probably contains many errors. Please refer any
questions or comments to the author below.

本文档试图描述如何使用Perl API，同时提供一些Perl核心基本功能的信息。本文档仍未完成且可能包含很多错误。如有问题或注释请向下面列出的作者提出。

#变量

=head1 Variables

##类型

=head2 Datatypes

Perl has three typedefs that handle Perl's three main data types:

Perl有三种typedef定义，用于处理Perl的三种主要数据类型。

* SV Scalar Value
* AV Array Value
* HV Hash Value

Each typedef has specific routines that manipulate the various data types.

每种类型都有相应的数据管理方式。

##什么是"IV"?

=head2 What is an "IV"?

Perl uses a special typedef IV which is a simple signed integer type that is
guaranteed to be large enough to hold a pointer (as well as an integer).
Additionally, there is the UV, which is simply an unsigned IV.

Perl使用typedef:'IV'(一个有符号整型，空间足以保存一个指针)。另外还有typedef:'UV'(一个无符号的'IV')。

Perl also uses two special typedefs, I32 and I16, which will always be at
least 32-bits and 16-bits long, respectively. (Again, there are U32 and U16,
as well.)  They will usually be exactly 32 and 16 bits long, but on Crays
they will both be 64 bits.

Perl同样适用I32和I16两个typedef，通常至少有32位或16位长(同样也有U32和U64)。但是在Crays上，它们都是64位。

##使用SV

=head2 Working with SVs

An SV can be created and loaded with one command.  There are five types of
values that can be loaded: an integer value (IV), an unsigned integer
value (UV), a double (NV), a string (PV), and another scalar (SV).
("PV" stands for "Pointer Value".  You might think that it is misnamed
because it is described as pointing only to strings.  However, it is
possible to have it point to other things.  For example, inversion
lists, used in regular expression data structures, are scalars, each
consisting of an array of UVs which are accessed through PVs.  But,
using it for non-strings requires care, as the underlying assumption of
much of the internals is that PVs are just for strings.  Often, for
example, a trailing NUL is tacked on automatically.  The non-string use
is documented only in this paragraph.)

一个SV可以用一条指令创建和加载。有五种类型的值可以被加载：整型值(IV)，无符号整型值(UV)，double(NV)，
字符串(PV)以及另一个scalar(SV)。("PV"代表"指针值",你也许认为这样的名字有误导性因为它被描述为一个指向字符串的指针。
然而它实际上可以指向其它类型的数据。例如，反向列表被用于正则表达式的数据结构中，就是scalars，每一个成员都包含一个通过PV访问的UV数组。
但是将PV用于字符串之外的地方需要特别小心，因为很多内部的封装都假设PV是用于字符串的。非字符串的应用文档中只在此处提及。)

The seven routines are:

这七种操作如下：

    SV*  newSViv(IV);
    SV*  newSVuv(UV);
    SV*  newSVnv(double);
    SV*  newSVpv(const char*, STRLEN);
    SV*  newSVpvn(const char*, STRLEN);
    SV*  newSVpvf(const char*, ...);
    SV*  newSVsv(SV*);

C<STRLEN\> is an integer type (Size\_t, usually defined as size_t in
F<config.h>) guaranteed to be large enough to represent the size of
any string that perl can handle.

STRLEN 是整型(Size_t, 通常在config.h文件中被定义为size_t)，保证足够大以容纳任何perl可以处理的字符串的长度。

In the unlikely case of a SV requiring more complex initialisation, you
can create an empty SV with newSV(len).  If C<len> is 0 an empty SV of
type NULL is returned, else an SV of type PV is returned with len + 1 (for
the NUL) bytes of storage allocated, accessible via SvPVX.  In both cases
the SV has the undef value.

在一些不太可能发生的场景中SV需要更加复杂的初始化方式，你可以用newSV(len)创建一个空的SV。如果len为0，一个类型为NULL的空SV将会返回，否则一个类型为PV的SV将会返回，并分配长度为len+1字节且值为NULL的内存空间，可以通过SvPVX访问。在以上两种场景中SV的值都是undef。

    SV *sv = newSV(0);   /* no storage allocated  */
    SV *sv = newSV(10);  /* 10 (+1) bytes of uninitialised storage
                            allocated */
To change the value of an I<already-existing\> SV, there are eight routines:

有八种操作用于改变（已存在的）SV的值。

    void  sv_setiv(SV*, IV);
    void  sv_setuv(SV*, UV);
    void  sv_setnv(SV*, double);
    void  sv_setpv(SV*, const char*);
    void  sv_setpvn(SV*, const char*, STRLEN)
    void  sv_setpvf(SV*, const char*, ...);
    void  sv_vsetpvfn(SV*, const char*, STRLEN, va_list *, SV **, I32, bool *);
    void  sv_setsv(SV*, SV*);

Notice that you can choose to specify the length of the string to be
assigned by using C<sv_setpvn>, C<newSVpvn>, or C<newSVpv>, or you may
allow Perl to calculate the length by using C<sv_setpv> or by specifying
0 as the second argument to C<newSVpv>.  Be warned, though, that Perl will
determine the string's length by using C<strlen>, which depends on the
string terminating with a NUL character, and not otherwise containing
NULs.

注意：你可以使用sv\_setpvn，newSVpvn，或newSVpv设置传入字符串的长度，也可以用sv_setpv或newSVpn（第二个参数传0）让Perl自己计算字符串的长度，Perl会使用strlen来获取字符串的长度，因此你传入的字符串必须以NUL结尾，且中间不能包含NUL。

The arguments of C<sv_setpvf> are processed like C<sprintf>, and the
formatted output becomes the value.

sv_setpvf参数的处理类似于sprintf，格式化后的数据将被赋予SV*。

C<sv_vsetpvfn> is an analogue of C<vsprintf>, but it allows you to specify
either a pointer to a variable argument list or the address and length of
an array of SVs.  The last argument points to a boolean; on return, if that
boolean is true, then locale-specific information has been used to format
the string, and the string's contents are therefore untrustworthy (see
L<perlsec>).  This pointer may be NULL if that information is not
important.  Note that this function requires you to specify the length of
the format.

sv_vsetpvfn与vsprintf相似，但是它允许你指定一个指向参数列表的指针或者一个SV数组的地址和它的长度。最后一个参数是指向一个boolean的地址；返回后，如果boolean为true，则本地化的信息将被用于格式化字符串。因此该字符串的内容是不可信的（参见perlsec）。如果该信息不重要的话这个指针的值可以为NUL。注意该函数要求你提供格式的长度。

The C<sv_set*()> functions are not generic enough to operate on values
that have "magic".  See L<Magic Virtual Tables> later in this document.

sc_set*系列函数在操作“有魔力的”值的时候并不通用。参见本文中Magic Vitrual Table一节。

All SVs that contain strings should be terminated with a NUL character.
If it is not NUL-terminated there is a risk of
core dumps and corruptions from code which passes the string to C
functions or system calls which expect a NUL-terminated string.
Perl's own functions typically add a trailing NUL for this reason.
Nevertheless, you should be very careful when you pass a string stored
in an SV to a C function or system call.

所有包含字符串的SVs都应当以NUL结尾。如果不这样的话将会有将非NUL结尾字符串传入需要通过NUL判断字符串结束的C函数或系统调用从而导致进程崩溃从而核心转储的风险。Perl自己的函数一般都会因此添加一个尾部的NUL。当你将一个存在SV中的字符串传入C函数或系统调用的时候一定要万分小心。

To access the actual value that an SV points to, you can use the macros:

为得到SV指向的真实值，你可以用以下宏：

    SvIV(SV*)
    SvUV(SV*)
    SvNV(SV*)
    SvPV(SV*, STRLEN len)
    SvPV_nolen(SV*)

which will automatically coerce the actual scalar type into an IV, UV, double,
or string.

它们将把scalar类型自动转为IV，UV，double或字符串。

In the C<SvPV> macro, the length of the string returned is placed into the
variable C<len> (this is a macro, so you do I<not> use C<&len>).  If you do
not care what the length of the data is, use the C<SvPV_nolen> macro.
Historically the C<SvPV> macro with the global variable C<PL_na> has been
used in this case.  But that can be quite inefficient because C<PL_na> must
be accessed in thread-local storage in threaded Perl.  In any case, rememberthat Perl allows arbitrary strings of data that may both contain NULs and might not be terminated by a NUL.

对于SvPV这个宏，返回字符串的长度将被存放于变量len中（这是一个宏，应此你不需要使用len的地址）。如果你不介意数据的长度，使用宏SvPV_nolen，历史上，SvPV宏结合PL_na全局变量被使用在该场景下。但是这样非常低效，因为PL_na的位置位于支持多线程的Perl中的线程专属内存中。请记住，在任何场景下Perl之支持包含NUL或不以NUL结尾的字符串。

Also remember that C doesn't allow you to safely say C<foo(SvPV(s, len),
len);>. It might work with your compiler, but it won't work for everyone.
Break this sort of statement up into separate assignments:

同时请记住C语言不允许你安全得使用foo(SvPV(s.len),len)。这样的用法可能对于你的编译器啃一正常编译，但不保证对所有人都适用。请将这种类型的语句拆分成多条赋值语句。

    SV *s;
    STRLEN len;
    char *ptr;
    ptr = SvPV(s, len);
    foo(ptr, len);

If you want to know if the scalar value is TRUE, you can use:

如果你想知道某个scalar值是否为TRUE，你可以使用：

    SvTRUE(SV*)

Although Perl will automatically grow strings for you, if you need to force
Perl to allocate more memory for your SV, you can use the macro

尽管Perl将会自动为你的字符串分配内存，但是如果你需要强制为SV分配更多的内存的话，使用宏：

    SvGROW(SV*, STRLEN newlen)

which will determine if more memory needs to be allocated.  If so, it will
call the function C<sv_grow>.  Note that C<SvGROW> can only increase, not
decrease, the allocated memory of an SV and that it does not automatically
add space for the trailing NUL byte (perl's own string functions typically do C<SvGROW(sv, len + 1)>).

这将会检测是否需要分配更多的内存，如果是，它将会调用函数sv_grow。请注意SvGROW只会增加内存，且分配的内存不会自动在末尾添加NUL字节（perl内部一般都使用 SvGROW(sv, len + 1)）。

If you have an SV and want to know what kind of data Perl thinks is stored
in it, you can use the following macros to check the type of SV you have.

如果你想知道Perl认为一个SV中存储数据的的类型，可以使用如下的宏进行检查。

    SvIOK(SV*)
    SvNOK(SV*)
    SvPOK(SV*)

You can get and set the current length of the string stored in an SV with
the following macros:

你可以使用以下宏获取当前存储在SV中的字符串的长度。

    SvCUR(SV*)
    SvCUR_set(SV*, I32 val)

You can also get a pointer to the end of the string stored in the SV
with the macro:

你同样可以使用以下宏获取指向SV中字符串末尾的指针。

    SvEND(SV*)

But note that these last three macros are valid only if C<SvPOK()> is true.

但是请注意最后三个宏只有在SvPOK返回真的时候才能合法使用。

If you want to append something to the end of string stored in an C<SV*>,
you can use the following functions:

如果你希望在SV*中储存的字符串后面添加数据，可以使用如下函数：

    void  sv_catpv(SV*, const char*);
    void  sv_catpvn(SV*, const char*, STRLEN);
    void  sv_catpvf(SV*, const char*, ...);
    void  sv_vcatpvfn(SV*, const char*, STRLEN, va_list *, SV **,
                                                             I32, bool);
    void  sv_catsv(SV*, SV*);

The first function calculates the length of the string to be appended by
using C<strlen>.  In the second, you specify the length of the string
yourself.  The third function processes its arguments like C<sprintf> and
appends the formatted output.  The fourth function works like C<vsprintf>.
You can specify the address and length of an array of SVs instead of the
va_list argument. The fifth function extends the string stored in the first
SV with the string stored in the second SV.  It also forces the second SV
to be interpreted as a string.

第一个函数使用strlen计算需要添加的字符串长度。第二个函数需要你自己制定字符串长度。第三个函数以类似于sprintf的方式格式化字符串并将结果添加到末尾。第四个函数类似于vsprintf，你可以指定一个SV数组的地址和长度以取代va_list参数。第五个函数用第二个SV*中的字符串拓展第一个参数。当然它会将第二个参数解析为字符串类型。

The C<sv_cat*()> functions are not generic enough to operate on values that
have "magic".  See L<Magic Virtual Tables> later in this document.

sv_cat*()系列的函数对于有"魔法的"值来说并不通用。请看下面的Magic Virtual Tables章节。

If you know the name of a scalar variable, you can get a pointer to its SV
by using the following:

如果你知道scalar变量的名字，你可以用以下这种方式获取到一个指向SV的指针。

    SV*  get_sv("package::varname", 0);

This returns NULL if the variable does not exist.

如果变量不存在的话将会返回NULL。

If you want to know if this variable (or any other SV) is actually C<defined>,
you can call:

如果你想知道某个变量是否被定义，你可以调用：

    SvOK(SV*)

The scalar C<undef> value is stored in an SV instance called C<PL_sv_undef>.

scalar值undef存在一个名字叫PL_sv_undef的SV实例中。

Its address can be used whenever an C<SV*> is needed. Make sure that
you don't try to compare a random sv with C<&PL_sv_undef>. For example
when interfacing Perl code, it'll work correctly for:

它的地址可以被用于任何一个以SV*为参数的地方。确保你不会用一个SV与&PL_sv_undef进行比较。例如当与Perl代码交互的时候，如下方可以正常工作：

  foo(undef);

But won't work when called as:

但是这样调用就无法工作了：

  $x = undef;
  foo($x);

So to repeat always use SvOK() to check whether an sv is defined.

所以请用SvOK()来检查sv是否被定义。

Also you have to be careful when using C<&PL_sv_undef> as a value in
AVs or HVs (see L<AVs, HVs and undefined values>).

同样当时使用&PL_sv_undef作为AVs或HVs的值的时候千万要小心（看AVs，HVs和undefined值那一章节）。

There are also the two values C<PL_sv_yes> and C<PL_sv_no>, which contain
boolean TRUE and FALSE values, respectively.  Like C<PL_sv_undef>, their
addresses can be used whenever an C<SV*> is needed.

同样PL_sv_yes和PL_sv_no是两个包含了布尔值TRUE和FALSE的值。和PL_sv_undef一样，它们的地址同样可以被用于任何期望SV*的地方。

Do not be fooled into thinking that C<(SV *) 0> is the same as C<&PL_sv_undef>.
Take this code:

不要认为(SV *)0和&PL_sv_undef是一样的，请看以下代码：

    SV* sv = (SV*) 0;
    if (I-am-to-return-a-real-value) {
            sv = sv_2mortal(newSViv(42));
    }
    sv_setsv(ST(0), sv);

This code tries to return a new SV (which contains the value 42) if it should
return a real value, or undef otherwise.  Instead it has returned a NULL
pointer which, somewhere down the line, will cause a segmentation violation,
bus error, or just weird results.  Change the zero to C<&PL_sv_undef> in the
first line and all will be well.

这段代码在应当返回实数的时候返回42，否则返回undef。但是它实际上返回的是一个NULL指针，
将会导致内存段冲突或错误或一些奇怪的情况。将第一行中的0修改为&PL_sv_undef会解决这个问题。

To free an SV that you've created, call C<SvREFCNT_dec(SV*)>.  Normally this
call is not necessary (see L<Reference Counts and Mortality>).

为了释放一个你创建的SV，调用SvREFCNT_dec(SV*)。通常你不需要调用这个方法（请看引用计数与销毁）。

## 偏移

=head2 Offsets

Perl provides the function C<sv_chop> to efficiently remove characters
from the beginning of a string; you give it an SV and a pointer to
somewhere inside the PV, and it discards everything before the
pointer. The efficiency comes by means of a little hack: instead of
actually removing the characters, C<sv_chop> sets the flag C<OOK>
(offset OK) to signal to other functions that the offset hack is in
effect, and it puts the number of bytes chopped off into the IV field
of the SV. It then moves the PV pointer (called C<SvPVX>) forward that
many bytes, and adjusts C<SvCUR> and C<SvLEN>.

Perl提供了函数sv_chop用于高效地移除字符串开头的字符；你传入一个SV和一个指向PV某处地址的指针，它将会移除指针前的全部字符。
它的效率源于一点hack技巧：它实际上并没有真正移除掉这些字符，sc_chop设置了OOK(offset OK)字段用于向其他函数表明使用了这种偏移的hack技巧，
同时它将被移除的字符的个数存入到SV中的IV字段。然后它将PV指针（称作SvPVX）向前移动那么多字节，并适配SvCUR和SvLEN。

Hence, at this point, the start of the buffer that we allocated lives
at C<SvPVX(sv) - SvIV(sv)> in memory and the PV pointer is pointing
into the middle of this allocated storage.

因此在这一点上，我们分配的缓冲区的起始点位于内存中的SvPVX(sv) - SvIV(sv)，而PV指针指向分配的内存中间的区域。

This is best demonstrated by example:

用下面这个例子可以很好地说明：

  % ./perl -Ilib -MDevel::Peek -le '$a="12345"; $a=~s/.//; Dump($a)'
  SV = PVIV(0x8128450) at 0x81340f0
    REFCNT = 1
    FLAGS = (POK,OOK,pPOK)
    IV = 1  (OFFSET)
    PV = 0x8135781 ( "1" . ) "2345"\0
    CUR = 4
    LEN = 5

Here the number of bytes chopped off (1) is put into IV, and
C<Devel::Peek::Dump> helpfully reminds us that this is an offset. The
portion of the string between the "real" and the "fake" beginnings is
shown in parentheses, and the values of C<SvCUR> and C<SvLEN> reflect
the fake beginning, not the real one.

这里被切除的字符序列长度1被存如IV中，而且Devel::Peek::Dump包提示我们这是一个偏移量。
在实际和伪起始位置之间的字符串显示在括号中，SvCUR和SvLEN表示的是伪起始位置，不是实际的值。

Something similar to the offset hack is performed on AVs to enable
efficient shifting and splicing off the beginning of the array; while
C<AvARRAY> points to the first element in the array that is visible from
Perl, C<AvALLOC> points to the real start of the C array. These are
usually the same, but a C<shift> operation can be carried out by
increasing C<AvARRAY> by one and decreasing C<AvFILL> and C<AvMAX>.
Again, the location of the real start of the C array only comes into
play when freeing the array. See C<av_shift> in F<av.c>.

与这种偏移的hack技巧类似的方法被应用于高效地移除或切除数组的开头元素；尽管AvARRAY指向的是Perl看来array的起始位置，
AvALLOC指向的才是数组的实际位置。它们通常都是一样的，但是shift操作是通过AvARRAY加1并且减少AvFILL和AvMAX的方式实现的。
数组实际的位置只有在需要释放内存的时候才会被用到。

## SV的结构

=head2 What's Really Stored in an SV?

Recall that the usual method of determining the type of scalar you have is
to use C<Sv\*OK> macros.  Because a scalar can be both a number and a string,
usually these macros will always return TRUE and calling the C<Sv*V>
macros will do the appropriate conversion of string to integer/double or
integer/double to string.

请回忆，为了判断SV的实际类型我们通常会使用Sv\*OK系列的宏。由于scalar可以同时是一个数字和字符串，
通常这些宏都会返回TRUE，并且调用调用Sv\*V系列的宏会自动对SV作出合适的转换。

If you I<really> need to know if you have an integer, double, or string
pointer in an SV, you can use the following three macros instead:

如果你真的需要知道SV中指向的究竟是一个整型，浮点型还是字符串，你可以使用下面三个宏代替：

    SvIOKp(SV*)
    SvNOKp(SV*)
    SvPOKp(SV*)

These will tell you if you truly have an integer, double, or string pointer
stored in your SV.  The "p" stands for private. 

这些方法可以检测SV中存储的究竟是整型，浮点型或者字符串指针。"p"代表private。

There are various ways in which the private and public flags may differ.
For example, a tied SV may have a valid underlying value in the IV slot
(so SvIOKp is true), but the data should be accessed via the FETCH
routine rather than directly, so SvIOK is false. Another is when
numeric conversion has occurred and precision has been lost: only the
private flag is set on 'lossy' values. So when an NV is converted to an
IV with loss, SvIOKp, SvNOKp and SvNOK will be set, while SvIOK wont be.

在很多情况下public和private标记是不同的。例如，一个tied的SV在IV槽中有一个合法的值（也就是说SvIOKp为true），
但是它的数据要通过FETCH方法简介获取，因此SvIOK为false。另一种情况是当发生了数字强转且丢失精度的时候：
只有private标记被设置为'lossy'。所以当NV被转为IV且丢失精度的时候，SvIOKp，SvNOKp和SvNOK都会被设置，而SvIOK不会。

In general, though, it's best to use the C<Sv*V> macros.

总而言之，最好使用Sv\*V系列的宏。

## 使用AVs

=head2 Working with AVs

There are two ways to create and load an AV.  The first method creates an
empty AV:

有两种方式可以创建并加载一个AV。第一种方法创建一个新的AV：

    AV*  newAV();

The second method both creates the AV and initially populates it with SVs:

第二种方法同时创建并用SVs初始化。

    AV*  av_make(I32 num, SV **ptr);

The second argument points to an array containing C<num> C<SV*>'s.  Once the
AV has been created, the SVs can be destroyed, if so desired.

第二个参数指向一个包含了num个SV*的指针。一旦AV被创建，Svs就可以被销毁，如果你希望这样的话。

Once the AV has been created, the following operations are possible on it:

一旦AV被创建，可以对其进行如下操作：

    void  av_push(AV*, SV*);
    SV*   av_pop(AV*);
    SV*   av_shift(AV*);
    void  av_unshift(AV*, I32 num);

These should be familiar operations, with the exception of C<av_unshift>.
This routine adds C<num> elements at the front of the array with the C<undef>
value.  You must then use C<av_store> (described below) to assign values
to these new elements.

这些都是你所熟悉的操作，除了av_unshift会在数组的前端添加num个值为undef的元素。你必须使用av_store为这些新元素赋值。

Here are some other functions:

这里还有其他一些函数：

    I32   av_top_index(AV*);
    SV**  av_fetch(AV*, I32 key, I32 lval);
    SV**  av_store(AV*, I32 key, SV* val);

The C<av_top_index> function returns the highest index value in an array (just
like $#array in Perl).  If the array is empty, -1 is returned.  The
C<av_fetch> function returns the value at index C<key>, but if C<lval>
is non-zero, then C<av_fetch> will store an undef value at that index.
The C<av_store> function stores the value C<val> at index C<key>, and does
not increment the reference count of C<val>.  Thus the caller is responsible
for taking care of that, and if C<av_store> returns NULL, the caller will
have to decrement the reference count to avoid a memory leak.  Note that
C<av_fetch> and C<av_store> both return C<SV**>'s, not C<SV*>'s as their
return value.

av_top_index返回数组的最大索引值（类似于Perl中的$#array）。如果数组为空则返回-1。av_fetch返回数组中索引为key的值，
但是如果lval非0，则av_fetch将会在那个索引处存入undef。av_store将在索引key处存入val，
并且不会增加val的引用计数。因此调用者需要处理它的索引计数，如果av_store返回NULL的话，调用者需要减少引用计数以防止内存泄漏。
注意，av_fetch和av_store都会返回SV**，而不是SV*作为它们的返回值。

A few more:

更多函数：

    void  av_clear(AV*);
    void  av_undef(AV*);
    void  av_extend(AV*, I32 key);

The C<av_clear> function deletes all the elements in the AV* array, but
does not actually delete the array itself.  The C<av_undef> function will
delete all the elements in the array plus the array itself.  The
C<av_extend> function extends the array so that it contains at least C<key+1>
elements.  If C<key+1> is less than the currently allocated length of the array,
then nothing is done.

av_clear将会删除AV*数组中所有的元素，但是并不会删除数组自身。av_undef将会删除数组的全部元素以及它自己。
av_extend函数将会拓展数组，因此它至少包含key+1个元素。如果key+1小于当前分配的数组长度，那么什么操作都不会进行。

If you know the name of an array variable, you can get a pointer to its AV
by using the following:

如果你知道数组的名字，你也可以这样获取一个指向这个AV的指针：

    AV*  get_av("package::varname", 0);

This returns NULL if the variable does not exist.

如果变量不存在的话将会返回NULL。

See L<Understanding the Magic of Tied Hashes and Arrays> for more
information on how to use the array access functions on tied arrays.

请看 '理解tied的hashes和arrays的魔法' 以获得更多或如何对tied array使用数组访问函数的信息。

## 使用HV

=head2 Working with HVs

To create an HV, you use the following routine:

你可以使用如下方法创建HV：

    HV*  newHV();

Once the HV has been created, the following operations are possible on it:

当HV被创建后，以下可以对其进行一下操作：

    SV**  hv_store(HV*, const char* key, U32 klen, SV* val, U32 hash);
    SV**  hv_fetch(HV*, const char* key, U32 klen, I32 lval);

The C<klen> parameter is the length of the key being passed in (Note that
you cannot pass 0 in as a value of C<klen> to tell Perl to measure the
length of the key).  The C<val> argument contains the SV pointer to the
scalar being stored, and C<hash> is the precomputed hash value (zero if
you want C<hv_store> to calculate it for you).  The C<lval> parameter
indicates whether this fetch is actually a part of a store operation, in
which case a new undefined value will be added to the HV with the supplied
key and C<hv_fetch> will return as if the value had already existed.

klen参数是你传入的key的长度（注意你不能传入0值，让Perl自己判断key的长度）。val参数存有一个指向需要存入的scalar的SV指针，
hash参数是预计算的hash值（如果你希望Perl为你算的话传入0）。lval参数决定了fetch操作是不是store操作的一部分，
在这种情况下一个新的undefined值会被加入到HV中指定key值的地方并且hv_fetch将会返回就好像这个值已经存在了一样。

Remember that C<hv_store> and C<hv_fetch> return C<SV**>'s and not just
C<SV*>.  To access the scalar value, you must first dereference the return
value.  However, you should check to make sure that the return value is
not NULL before dereferencing it.

记住hv_store和hv_getch返回SV**类型的值而不是SV*，为了获取scalar值，你必须先对返回值解引用。
然而，你应该先检查返回值非NULL再解引用。

The first of these two functions checks if a hash table entry exists, and the 
second deletes it.

下面这两个函数，第一个检查一个哈希表中是否存在某个键值，第二个删除它。

    bool  hv_exists(HV*, const char* key, U32 klen);
    SV*   hv_delete(HV*, const char* key, U32 klen, I32 flags);

If C<flags> does not include the C<G_DISCARD> flag then C<hv_delete> will
create and return a mortal copy of the deleted value.

如果G_DEICAED标记不存在则hv_delete将会创建并返回一个被删除值的拷贝。

And more miscellaneous functions:

更多函数：

    void   hv_clear(HV*);
    void   hv_undef(HV*);

Like their AV counterparts, C<hv_clear> deletes all the entries in the hash
table but does not actually delete the hash table.  The C<hv_undef> deletes
both the entries and the hash table itself.

和针对AV操作的其他对应函数一样，hv_clear删除哈希表中所有的键值，但不会删除哈希表。hv_undef同时删除键值和表。

Perl keeps the actual data in a linked list of structures with a typedef of HE.
These contain the actual key and value pointers (plus extra administrative
overhead).  The key is a string pointer; the value is an C<SV*>.  However,
once you have an C<HE*>, to get the actual key and value, use the routines
specified below.

Perl将实际的值保存在使用HE typedef的结构体链表中。它们包含了实际的键和指向值的指针（以及其他用于管理的头部）。
键是一个字符串指针；值是一个SV\*。然而，当你拥有一个HE*时，使用如下函数获取实际的键值：

    I32    hv_iterinit(HV*);
            /* Prepares starting point to traverse hash table */
    HE*    hv_iternext(HV*);
            /* Get the next entry, and return a pointer to a
               structure that has both the key and value */
    char*  hv_iterkey(HE* entry, I32* retlen);
            /* Get the key from an HE structure and also return
               the length of the key string */
    SV*    hv_iterval(HV*, HE* entry);
            /* Return an SV pointer to the value of the HE
               structure */
    SV*    hv_iternextsv(HV*, char** key, I32* retlen);
            /* This convenience routine combines hv_iternext,
	       hv_iterkey, and hv_iterval.  The key and retlen
	       arguments are return values for the key and its
	       length.  The value is returned in the SV* argument */

If you know the name of a hash variable, you can get a pointer to its HV
by using the following:

如果你知道哈希变量的名字，你可以用如下方法获取指向这个HV的指针：

    HV*  get_hv("package::varname", 0);

This returns NULL if the variable does not exist.

如果这个变量不存则会返回NULL。

The hash algorithm is defined in the C<PERL_HASH> macro:

哈希算法是通过PERL_HASH宏定义的：

    PERL_HASH(hash, key, klen)

The exact implementation of this macro varies by architecture and version
of perl, and the return value may change per invocation, so the value
is only valid for the duration of a single perl process.

这个宏的实现与架构和perl的版本有关，它的返回值每次调用都可能不一样，所以它的值只有在某一个perl进程内有效。

See L<Understanding the Magic of Tied Hashes and Arrays> for more
information on how to use the hash access functions on tied hashes.

请看 '理解tied的哈希和数组的魔法' 这一节以获取更多关于如何使用哈希访问函数访问tied哈希的信息。

## 哈希拓展API

=head2 Hash API Extensions

Beginning with version 5.004, the following functions are also supported:

自从5.004版本开始支持以下函数：

    HE*     hv_fetch_ent  (HV* tb, SV* key, I32 lval, U32 hash);
    HE*     hv_store_ent  (HV* tb, SV* key, SV* val, U32 hash);

    bool    hv_exists_ent (HV* tb, SV* key, U32 hash);
    SV*     hv_delete_ent (HV* tb, SV* key, I32 flags, U32 hash);

    SV*     hv_iterkeysv  (HE* entry);

Note that these functions take C<SV*> keys, which simplifies writing
of extension code that deals with hash structures.  These functions
also allow passing of C<SV*> keys to C<tie> functions without forcing
you to stringify the keys (unlike the previous set of functions).

注意，这些函数使用SV\*类型的键，这样简化了需要处理哈希结构的代码。这些函数同样允许传递SV\*类型的键给tie函数而不需要你将键强转为字符串（和之前的那些函数不同）。

They also return and accept whole hash entries (C<HE*>), making their
use more efficient (since the hash number for a particular string
doesn't have to be recomputed every time).  See L<perlapi> for detailed
descriptions.

它们同样返回并接受完整的哈希键值结构（HE*），使得它们更加高效（因为特定字符串的哈希值不用每次都重复计算）。请看perlapi以了解更多细节。

The following macros must always be used to access the contents of hash
entries.  Note that the arguments to these macros must be simple
variables, since they may get evaluated more than once.  See
L<perlapi> for detailed descriptions of these macros.

以下宏只能用于获取哈希键值结构的内容。注意这些宏的参数只能是简单的变量，因为它们可能会被计算多次。请看perlapi以了解更多细节。

    HePV(HE* he, STRLEN len)
    HeVAL(HE* he)
    HeHASH(HE* he)
    HeSVKEY(HE* he)
    HeSVKEY_force(HE* he)
    HeSVKEY_set(HE* he, SV* sv)

These two lower level macros are defined, but must only be used when
dealing with keys that are not C<SV*>s:

这两个底层的宏只能被用于键不是SV*的情况。

    HeKEY(HE* he)
    HeKLEN(HE* he)

Note that both C<hv_store> and C<hv_store_ent> do not increment the
reference count of the stored C<val>, which is the caller's responsibility.
If these functions return a NULL value, the caller will usually have to
decrement the reference count of C<val> to avoid a memory leak.

请注意hv_store和hv_store_ent都不会增加val的引用计数，这都是调用者需要做的。如果这些函数返回NULL值，调用者必须要减少引用计数的值以避免内存泄漏。

## AVs，HVs和undefined值

=head2 AVs, HVs and undefined values

Sometimes you have to store undefined values in AVs or HVs. Although
this may be a rare case, it can be tricky. That's because you're
used to using C<&PL_sv_undef> if you need an undefined SV.

有时你需要在AVs和HVs中存储undefined值。尽管这样的情形很少，但是却会很诡异。因为如果你需要一个未定义SV的时候得用&PV_sv_undef。

For example, intuition tells you that this XS code:

例如，直觉会告诉你以下XS代码：

    AV *av = newAV();
    av_store( av, 0, &PL_sv_undef );

is equivalent to this Perl code:

与以下代码等价：

    my @av;
    $av[0] = undef;

Unfortunately, this isn't true. AVs use C<&PL_sv_undef> as a marker
for indicating that an array element has not yet been initialized.
Thus, C<exists $av[0]> would be true for the above Perl code, but
false for the array generated by the XS code.

不幸的是这不是真的。AV使用&PL_sv_undef作为元素没有被初始化的标识。应此exists $av[0]对于Perl代码来说是对的，但是对于XS来说却返回假。

Other problems can occur when storing C<&PL_sv_undef> in HVs:

当在HV中存储&PL_sv_undef的时候也会出问题：

    hv_store( hv, "key", 3, &PL_sv_undef, 0 );

This will indeed make the value C<undef>, but if you try to modify
the value of C<key>, you'll get the following error:

这样的确会使值为undef，但是当你试图改变key对应的值的时候将会得到以下错误：

    Modification of non-creatable hash value attempted

    试图改变没有被创建的哈希值

In perl 5.8.0, C<&PL_sv_undef> was also used to mark placeholders
in restricted hashes. This caused such hash entries not to appear
when iterating over the hash or when checking for the keys
with the C<hv_exists> function.

在Perl5.8.0中，&PL_sv_undef也被用于在'受限'哈希表中标记占位符。
这样会导致哈希键值在遍历哈希表或使用hv_exists的时候不会出现。

You can run into similar problems when you store C<&PL_sv_yes> or
C<&PL_sv_no> into AVs or HVs. Trying to modify such elements
will give you the following error:

当你将&PL_sv_yes或&PL_sv_no存入AV或HV的时候会遇到类似的问题。试图去改变这些元素的时候会产生如下错误：

    Modification of a read-only value attempted

    试图改变只读类型的值

To make a long story short, you can use the special variables
C<&PL_sv_undef>, C<&PL_sv_yes> and C<&PL_sv_no> with AVs and
HVs, but you have to make sure you know what you're doing.

长话短说，你可以将&PL_sv_undef，&PL_sv_yes或&PL_sv_no这些特殊变量用户AV和HV，
但是请确保你知道自己在做什么。

Generally, if you want to store an undefined value in an AV
or HV, you should not use C<&PL_sv_undef>, but rather create a
new undefined value using the C<newSV> function, for example:

如果你想在AV或HV中存储一个undef值的时候，你不应该使用&PL_sv_undef，
而是用newSV创建一个新的未定义值，例如：

    av_store( av, 42, newSV(0) );
    hv_store( hv, "foo", 3, newSV(0), 0 );

## 引用

=head2 References

References are a special type of scalar that point to other data types
(including other references).

引用是一种特殊的scalar类型，用于指向其他的数据类型（包括引用类型）。

To create a reference, use either of the following functions:

请使用下列函数创建一个引用：

    SV* newRV_inc((SV*) thing);
    SV* newRV_noinc((SV*) thing);

The C<thing> argument can be any of an C<SV*>, C<AV*>, or C<HV*>.  The
functions are identical except that C<newRV_inc> increments the reference
count of the C<thing>, while C<newRV_noinc> does not.  For historical
reasons, C<newRV> is a synonym for C<newRV_inc>.

参数thing可以是SV*，AV\*或HV*中的任意一种。这两个函数除了newRV_inc会增加thing的引用计数外，其他方面都是完全一样。newTV就是newRV_inc的匿名方法（某些历史原因导致的）。

Once you have a reference, you can use the following macro to dereference
the reference:

当你有一个引用的时候，你可以使用下列宏来对其解引用：

    SvRV(SV*)

then call the appropriate routines, casting the returned C<SV*> to either an
C<AV*> or C<HV*>, if required.

然后调用合适的方法，将返回的SV*强转为AV\*或HV*。

To determine if an SV is a reference, you can use the following macro:

可以用下面的宏判断一个SV是否为一个引用。

    SvROK(SV*)

To discover what type of value the reference refers to, use the following
macro and then check the return value.

为了发现引用指向的类型，使用下列宏然后检查返回值。

    SvTYPE(SvRV(SV*))

The most useful types that will be returned are:

最有用的返回类型有如下几种：

    SVt_PVAV    Scalar
    SVt_PVAV    Array
    SVt_PVHV    Hash
    SVt_PVCV    Code
    SVt_PVGV    Glob (possibly a file handle)

See L<perlapi/svtype> for more details.

请参开 perlapi/svtype 以获得更多细节。

## 被blessed的引用和类对象

=head2 Blessed References and Class Objects

References are also used to support object-oriented programming.  In perl's
OO lexicon, an object is simply a reference that has been blessed into a
package (or class).  Once blessed, the programmer may now use the reference
to access the various methods in the class.

引用同样可以被用于支持面向对象编程。在Perl的OO语法中，一个对象就是一个被blessed到包（或类）之中的引用。一旦被blessed，程序员就可以使用这个引用访问类的各种方法。

A reference can be blessed into a package with the following function:

一个引用可以通过使用如下方法被blessed到包中：

    SV* sv_bless(SV* sv, HV* stash);

The C<sv> argument must be a reference value.  The C<stash> argument
specifies which class the reference will belong to.  See
L<Stashes and Globs> for information on converting class names into stashes.

参数sv必须为引用类型。参数stash指定了引用所属的类。查看 'Stashes 和 Globs'章节 以获的更多关于将类名转为stash的信息。

/* Still under construction */

/* 仍在建设中 */

The following function upgrades rv to reference if not already one.
Creates a new SV for rv to point to.  If C<classname> is non-null, the SV
is blessed into the specified class.  SV is returned.

下列函数将参数rv升级为引用。使得rv指向一个新创建的SV。如果classname不是NULL的话，SV将会被blessed成制定的类。将返回这个SV。

	SV* newSVrv(SV* rv, const char* classname);

The following three functions copy integer, unsigned integer or double
into an SV whose reference is C<rv>.  SV is blessed if C<classname> is
non-null.

下面三个函数拷贝整型，无符号整型或浮点型到一个引用为rv的SV中。如果classname非空的话SV将被blessed。

	SV* sv_setref_iv(SV* rv, const char* classname, IV iv);
	SV* sv_setref_uv(SV* rv, const char* classname, UV uv);
	SV* sv_setref_nv(SV* rv, const char* classname, NV iv);

The following function copies the pointer value (I<the address, not the
string!>) into an SV whose reference is rv.  SV is blessed if C<classname>
is non-null.

下面的函数拷贝指针的值到rv引用指向的SV里。如果classname非空的话SV将被blessed。

	SV* sv_setref_pv(SV* rv, const char* classname, void* pv);

The following function copies a string into an SV whose reference is C<rv>.
Set length to 0 to let Perl calculate the string length.  SV is blessed if
C<classname> is non-null.

下面的函数将字符串拷贝到rv指向的SV里，当length为0的时候Perl会自动计算字符串的长度。如果classname非空的话SV将被blessed。

    SV* sv_setref_pvn(SV* rv, const char* classname, char* pv,
                                                         STRLEN length);

The following function tests whether the SV is blessed into the specified
class.  It does not check inheritance relationships.

下面的函数测试SV是否被blessed成某种特定的类。它不会检查继承关系。

	int  sv_isa(SV* sv, const char* name);

The following function tests whether the SV is a reference to a blessed object.

下面的函数测试SV是否为一个被blessed过的引用。

	int  sv_isobject(SV* sv);

The following function tests whether the SV is derived from the specified
class. SV can be either a reference to a blessed object or a string
containing a class name. This is the function implementing the
C<UNIVERSAL::isa> functionality.

下面的函数测试SV是否继承自某个特定的类。SV既可以是一个被blessed过的引用也可以是包含类名的字符串。
就是这个函数实现了UNIVERSAL::ISA的功能。

	bool sv_derived_from(SV* sv, const char* name);

To check if you've got an object derived from a specific class you have
to write:

为了检查你是否得到了一个继承自特定类的对象，请使用：

	if (sv_isobject(sv) && sv_derived_from(sv, class)) { ... }

## 创建新变量

=head2 Creating New Variables

To create a new Perl variable with an undef value which can be accessed from
your Perl script, use the following routines, depending on the variable type.

为了创建一个你可以用脚本访问到的值为undef的新的Perl变量，根据变量的类型使用以下函数：

    SV*  get_sv("package::varname", GV_ADD);
    AV*  get_av("package::varname", GV_ADD);
    HV*  get_hv("package::varname", GV_ADD);

Notice the use of GV_ADD as the second parameter.  The new variable can now
be set, using the routines appropriate to the data type.

请注意第二个参数GC_ADD。用合适的方法就可以设置合适类型的变量。

There are additional macros whose values may be bitwise OR'ed with the
C<GV_ADD> argument to enable certain extra features.  Those bits are:

也有额外的宏，其值会被用于和GV_ADD相或以开启新的特性。这些宏如下：

=over

=item GV_ADDMULTI

* GV_ADDMULTI

Marks the variable as multiply defined, thus preventing the:

标记这个变量是多重定义的，应此避免了：

    Name <varname> used only once: possible typo

warning.

    名称为 'varname' 的变量只适用了一次：可能为打字错误。

的警告。

=item GV_ADDWARN

* GV_ADDWARN

Issues the warning:

表明了如下警告：

    Had to create <varname> unexpectedly

    if the variable did not exist before the function was called.

-

    如果变量名为varname的变量在函数调用时不存在的时候必须要创建它。

=back

If you do not specify a package name, the variable is created in the current
package.

如果你没有指定包名的的阿虎，变量将会创建在在当前包中。

## 引用计数和销毁

=head2 Reference Counts and Mortality

Perl uses a reference count-driven garbage collection mechanism. SVs,
AVs, or HVs (xV for short in the following) start their life with a
reference count of 1.  If the reference count of an xV ever drops to 0,
then it will be destroyed and its memory made available for reuse.

Perl使用引用计数的垃圾收集机制。在SV，AV或HV（以下用xV代替）生命周期的开始它们的引用计数被初始化为1。如果xV的引用计数降低到0，它将会被销毁以释放内存。

This normally doesn't happen at the Perl level unless a variable is
undef'ed or the last variable holding a reference to it is changed or
overwritten.  At the internal level, however, reference counts can be
manipulated with the following macros:

这通常不会发生在Perl语言的层面，除非一个变量被执行undef操作或者最有一个保存引用的变量被修改或覆盖。在Perl内部实现的层面可以通过以下宏管理引用计数。

    int SvREFCNT(SV* sv);
    SV* SvREFCNT_inc(SV* sv);
    void SvREFCNT_dec(SV* sv);

However, there is one other function which manipulates the reference
count of its argument.  The C<newRV_inc> function, you will recall,
creates a reference to the specified argument.  As a side effect,
it increments the argument's reference count.  If this is not what
you want, use C<newRV_noinc> instead.

还有一个可以管理其参数引用计数的函数。newRV_inc可以创建它参数的一个引用。它的副作用是增加其参数的引用计数。如果这不是你想要的效果，使用newRV_noinc来代替。

For example, imagine you want to return a reference from an XSUB function.
Inside the XSUB routine, you create an SV which initially has a reference
count of one.  Then you call C<newRV_inc>, passing it the just-created SV.
This returns the reference as a new SV, but the reference count of the
SV you passed to C<newRV_inc> has been incremented to two.  Now you
return the reference from the XSUB routine and frget about the SV.
But Perl hasn't!  Whenever the returned reference is destroyed, the
reference count of the original SV is decreased to one and nothing happens.
The SV will hang around without any way to access it until Perl itself
terminates.  This is a memory leak.

例如，想象一下你想要从XSUB函数返回一个引用。在XSUB函数内，你创建了一个引用计数为1的SV。然后你调用了newRV_inc，传入刚刚创建的SV，它会将对SV的引用作为一个新的SV返回。此时你传入的SV的引用计数已经是2了。现在你将这个引用从XSUB函数中返回并且忘掉了传入的那个SV。但是Perl没有！当返回的引用被销毁有，原先那个SV的引用计数变成1，什么都不会发生。这个SV将会一直存在直到Perl进程终止。这就是一个内存泄漏。

The correct procedure, then, is to use C<newRV_noinc> instead of
C<newRV_inc>.  Then, if and when the last reference is destroyed,
the reference count of the SV will go to zero and it will be destroyed,
stopping any memory leak.

正确的做法是使用newRV_noinc取代newRV_inc。然后如果最后一个引用被销毁，SV的引用计数变为0它将会被销毁，避免了内存泄漏。

There are some convenience functions available that can help with the
destruction of xVs.  These functions introduce the concept of "mortality".
An xV that is mortal has had its reference count marked to be decremented,
but not actually decremented, until "a short time later".  Generally the
term "short time later" means a single Perl statement, such as a call to
an XSUB function.  The actual determinant for when mortal xVs have their
reference count decremented depends on two macros, SAVETMPS and FREETMPS.
See L<perlcall> and L<perlxs> for more details on these macros.

还有一些更加方便的函数可以帮助你销毁xV。这些函数引入了"死亡性"这个概念。
一个xV是"将死的"如果它的引用计数被标记为"减少的"，但是并没有实际减少而是"延迟到之后再减少"。一般来说"延迟之后"意味着一条Perl语句，例如调用一个XSUB函数。"将死"的xV的引用计数实际减少的时机是由两个宏决定的，SAVETMPS和FREETMPS。请看perlcall和perlxs以获得更多关于这些宏的细节。

"Mortalization" then is at its simplest a deferred C<SvREFCNT_dec>.
However, if you mortalize a variable twice, the reference count will
later be decremented twice.

"死亡性"可以理解为一个延时的SvREFCNT_dec。如果你'将死'一个变量两次，引用计数将会在之后减少两次。


"Mortal" SVs are mainly used for SVs that are placed on perl's stack.
For example an SV which is created just to pass a number to a called sub
is made mortal to have it cleaned up automatically when it's popped off
the stack. Similarly, results returned by XSUBs (which are pushed on the
stack) are often made mortal.

"将死"的SV通常被用于那些被分配在perl的栈上的SV。例如一个被创建用于传入一个数字到到函数中的SV，将被设置为'将死的'以使其在被栈弹出时自动释放。同样的，XSUB返回的结果（被放置到栈上）通常也会被设置为"将死的"。

To create a mortal variable, use the functions:

为了创建一个"将死的"变量，使用如下函数：

    SV*  sv_newmortal()
    SV*  sv_2mortal(SV*)
    SV*  sv_mortalcopy(SV*)

The first call creates a mortal SV (with no value), the second converts an existing
SV to a mortal SV (and thus defers a call to C<SvREFCNT_dec>), and the
third creates a mortal copy of an existing SV.
Because C<sv_newmortal> gives the new SV no value, it must normally be given one
via C<sv_setpv>, C<sv_setiv>, etc. :

第一个调用创建一个没有值的将死SV，第二个函数将一个已经存在的SV转为将死的SV（也就是说延迟了一次SvREFCNT_dec的调用），第三个函数创建了一个已存在SV的将死SV拷贝。
因为sv_newmortal没有给新建的SV设置任何值，它通常必须要用sv_setpv或sv_setiv等设置一个值:

    SV *tmp = sv_newmortal();
    sv_setiv(tmp, an_integer);

As that is multiple C statements it is quite common so see this idiom instead:

那是写成多条C语句，通常以下用法也经常可以看到。

    SV *tmp = sv_2mortal(newSViv(an_integer));


You should be careful about creating mortal variables.  Strange things
can happen if you make the same value mortal within multiple contexts,
or if you make a variable mortal multiple times. Thinking of "Mortalization"
as deferred C<SvREFCNT_dec> should help to minimize such problems.
For example if you are passing an SV which you I<know> has a high enough REFCNT
to survive its use on the stack you need not do any mortalization.
If you are not sure then doing an C<SvREFCNT_inc> and C<sv_2mortal>, or
making a C<sv_mortalcopy> is safer.

在创建"将死"变量的时候要非常仔细。如果你在多个场景下将同一个变量设置为'将死'或将一个变量设置多次，
可能会导致奇怪的事情发生。将"将死性"想象成延迟的Sv_REFCNT_dec可以帮之你减少这样的问题。
例如如果你传入一个引用计数足够大（可以在栈上存活）的SV，就不需要设置为'将死'了。如果你不确定那么可以先做一次SvREFCNT_inc操作在调用sv_2mortal，
或者使用sv_mortalcopy将会更加安全。

The mortal routines are not just for SVs; AVs and HVs can be
made mortal by passing their address (type-casted to C<SV*>) to the
C<sv_2mortal> or C<sv_mortalcopy> routines.

'将死化'函数不仅仅可用于SV；AV和HV也可以通过将它们的地址（强转为类型SV*）传入sv_2mortal或sv_mortalcopy方法以设置为'将死的';

## Stashes和Globs

=head2 Stashes and Globs

A B<stash> is a hash that contains all variables that are defined
within a package.  Each key of the stash is a symbol
name (shared by all the different types of objects that have the same
name), and each value in the hash table is a GV (Glob Value).  This GV
in turn contains references to the various objects of that name,
including (but not limited to) the following:

一个stash是一个包括了一个包里面全部变量定义的哈希。stash中的每一个键都是一个符号名（被所有不同类型的同名对象共享），哈希表中的每一个值都是一个GV（Glob Value）。
这个GV同样包含了各种叫那个名字的对象的引用，包括（但不限于）以下几种：

    Scalar Value
    Array Value
    Hash Value
    I/O Handle
    Format
    Subroutine

There is a single stash called C<PL_defstash> that holds the items that exist
in the C<main> package.  To get at the items in other packages, append the
string "::" to the package name.  The items in the C<Foo> package are in
the stash C<Foo::> in PL_defstash.  The items in the C<Bar::Baz> package are
in the stash C<Baz::> in C<Bar::>'s stash.

有一个称作PL_defstash的stash存储了main包中的全部成员。要得到其他包中的成员，在包名后添加"::"。
Foo包中的成员在PL_defstash中名为Foo::的stash中。Bar::Baz包中的成员在Bar::的stash中的Baz::的stash中。

To get the stash pointer for a particular package, use the function:

为了得到特定包的stash执政，使用下列函数：

    HV*  gv_stashpv(const char* name, I32 flags)
    HV*  gv_stashsv(SV*, I32 flags)

The first function takes a literal string, the second uses the string stored
in the SV.  Remember that a stash is just a hash table, so you get back an
C<HV*>.  The C<flags> flag will create a new package if it is set to GV_ADD.

第一个函数需要传入一个字面字符串，第二个使用SV中储存的字符串。记住stash仅仅是一个哈希表，因此你可以获取一个HV*返回值。
当flags标记被设置为GV_ADD的时候将会创建一个新包。

The name that C<gv_stash*v> wants is the name of the package whose symbol table
you want.  The default package is called C<main>.  If you have multiply nested
packages, pass their names to C<gv_stash*v>, separated by C<::> as in the Perl
language itself.

gv_stash*v系列函数需要的参数name就是你需要的符号表所属的包的名字。默认的包名为main。如果你有多重嵌套的包，请用::将它们分割传入，就像在Perl语言中所做的那样。

Alternately, if you have an SV that is a blessed reference, you can find
out the stash pointer by using:

另外，如果你有一个被blessed的引用，你可以这样获取它的stash指针：

    HV*  SvSTASH(SvRV(SV*));

then use the following to get the package name itself:

然后这样获取它的包名：

    char*  HvNAME(HV* stash);

If you need to bless or re-bless an object you can use the following
function:

如果你需要bless或重复bless一个对象你可以用以下函数：

    SV*  sv_bless(SV*, HV* stash)

where the first argument, an C<SV*>, must be a reference, and the second
argument is a stash.  The returned C<SV*> can now be used in the same way
as any other SV.

第一个参数SV\*必须是引用类型，第二次参数是一个stash。返回的SV\*可以像其他任何SV一样使用。

For more information on references and blessings, consult L<perlref>.

参考prelref以获取关于引用和bless的更多信息。

## 双类型SV

=head2 Double-Typed SVs

Scalar variables normally contain only one type of value, an integer,
double, pointer, or reference.  Perl will automatically convert the
actual scalar data from the stored type into the requested type.

Scalar变量一般只包含一种类型的值，一个整型，一个浮点型，指针或引用。Perl很自动将实际的scalar数据从存储的类型转为需要的类型。

Some scalar variables contain more than one type of scalar data.  For
example, the variable $! contains either the numeric value of C<errno>
or its string equivalent from either strerror or sys_errlist[].

一些scalar变来那个包含一种以上类型的scalar数据。例如，变量$!含了errno的数字值或它的等价字符串（来源于strerror或sys_errlist[]）

To force multiple data values into an SV, you must do two things: use the
sv_set*v routines to add the additional scalar type, then set a flag
so that Perl will believe it contains more than one type of data.  The
four macros to set the flags are:

为了将多个数据存入一个SV，你必须做两件事：用sv_set*v方法添加额外的scalar类型，然后设置一个标记使得Perl认为它包好一种以上类型的数据。
这四个设置标记的宏如下：

	SvIOK_on
	SvNOK_on
	SvPOK_on
	SvROK_on

The particular macro you must use depends on which sv_set*v routine
you called first.  This is because every C<sv_set*v> routine turns on
only the bit for the particular type of data being set, and turns off
all the rest.

究竟使用哪一种宏取决于你调用哪一个sv_set*v方法。这是因为每一个sv_set*v方法都会打开所属数据类型的标记并且关闭其他所有标记。

For example, to create a new Perl variable called "dberror" that contains
both the numeric and descriptive string error values, you could use the
following code:

例如为了创建一个名叫dberror的变量，它既包含错误码的数字也包含描述字符串，你可以使用以下代码：

    extern int  dberror;
    extern char *dberror_list;

    SV* sv = get_sv("dberror", GV_ADD);
    sv_setiv(sv, (IV) dberror);
    sv_setpv(sv, dberror_list[dberror]);
    SvIOK_on(sv);

If the order of C<sv_setiv> and C<sv_setpv> had been reversed, then the
macro C<SvPOK_on> would need to be called instead of C<SvIOK_on>.

如果sv_setiv和sv_setpv的顺序颠倒了，那么你应该使用SvPOK_on这个宏代替SvIOK_on。

## 魔法变量

=head2 Magic Variables

[This section still under construction.  Ignore everything here.  Post no
bills.  Everything not permitted is forbidden.]

[这一节仍在建设中。请忽略这里的内容。任何未被允许的行为都是禁止的。]

Any SV may be magical, that is, it has special features that a normal
SV does not have.  These features are stored in the SV structure in a
linked list of C<struct magic>'s, typedef'ed to C<MAGIC>.

任何SV都可能是有魔力的，也就是说它具备一般SV所没有的特性。这些特性被存储在SV结构体中的struct magic链表中，被typedef为MAGIC。

    struct magic {
        MAGIC*      mg_moremagic;
        MGVTBL*     mg_virtual;
        U16         mg_private;
        char        mg_type;
        U8          mg_flags;
        I32         mg_len;
        SV*         mg_obj;
        char*       mg_ptr;
    };

Note this is current as of patchlevel 0, and could change at any time.

注意这个结构定义处于补丁阶段0，将来随时可能会发生改变。

## 设置魔法属性

=head2 Assigning Magic

Perl adds magic to an SV using the sv_magic function:

Perl使用sv_magic函数为SV添加魔法性：

  void sv_magic(SV* sv, SV* obj, int how, const char* name, I32 namlen);

The C<sv> argument is a pointer to the SV that is to acquire a new magical
feature.

参数sv是一个指向需要新的魔法特性的SV的指针。

If C<sv> is not already magical, Perl uses the C<SvUPGRADE> macro to
convert C<sv> to type C<SVt_PVMG>. Perl then continues by adding new magic
to the beginning of the linked list of magical features.  Any prior entry
of the same type of magic is deleted.  Note that this can be overridden,
and multiple instances of the same type of magic can be associated with an
SV.

如果sv当前是没有魔法性的，Perl使用SvUPGRADE宏将sv转型为SVt_PVMG。Perl然后继续添加新的魔法到魔法特性链表的最前端。
任何之前相同类型的魔法特性会被删除。注意这种特性是可以被覆盖的，多个相同类型的魔法特性可以被关联到同一个sv上。

The C<name> and C<namlen> arguments are used to associate a string with
the magic, typically the name of a variable. C<namlen> is stored in the
C<mg_len> field and if C<name> is non-null then either a C<savepvn> copy of
C<name> or C<name> itself is stored in the C<mg_ptr> field, depending on
whether C<namlen> is greater than zero or equal to zero respectively.  As a
special case, if C<(name && namlen == HEf_SVKEY)> then C<name> is assumed
to contain an C<SV*> and is stored as-is with its REFCNT incremented.

name和namlen参数被用于为这种魔法特性关联一个字符串，一般是变量名。namlen储存在mg_len字段。
如果参数name非空，那么一个name的savepvn拷贝或者name自身将会被存储在mg_ptr字段，这取决于namlen是大于0还是等于0。
当name&&namlen == HEf_SVKEY这种特殊情况下，name被认为包含一个SV*且被保存时其引用计数加一。

The sv_magic function uses C<how> to determine which, if any, predefined
"Magic Virtual Table" should be assigned to the C<mg_virtual> field.
See the L<Magic Virtual Tables> section below.  The C<how> argument is also
stored in the C<mg_type> field. The value of C<how> should be chosen
from the set of macros C<PERL_MAGIC_foo> found in F<perl.h>. Note that before
these macros were added, Perl internals used to directly use character
literals, so you may occasionally come across old code or documentation
referring to 'U' magic rather than C<PERL_MAGIC_uvar> for example.

sv_magic函数使用参数how以决定哪一个预定义的"魔法虚表"项将会被赋予mg_virtual字段。
请看下面 '魔法虚表' 一节。how参数同样存在mg_type字段。how的值应该取自PERL_MAGIC_foo系列的宏（定义在perl.h）。
注意在这些宏被添加以前，Perl内部是直接使用字面字符的，例如你偶尔会看到在一些旧的代码或文档中会使用'U'魔法而不是PERL_MAGIC_uvar。

The C<obj> argument is stored in the C<mg_obj> field of the C<MAGIC>
structure.  If it is not the same as the C<sv> argument, the reference
count of the C<obj> object is incremented.  If it is the same, or if
the C<how> argument is C<PERL_MAGIC_arylen>, or if it is a NULL pointer,
then C<obj> is merely stored, without the reference count being incremented.

obj参数存在MAGIC结构体的mg_obj字段。如果他和sv参数不同的话，obj对象的引用计数将会增加。如果是相同的或者参数how是PERL_MAGIC_arylen或者它就是一个空指针，
那么obj就仅仅储存在那，而不会增加引用计数。

See also C<sv_magicext> in L<perlapi> for a more flexible way to add magic
to an SV.

请看perlapi中的sv_magicext一节，那里有跟多更加灵活的向SV添加魔法的方式。

There is also a function to add magic to an C<HV>:

同样有一个用来向HV添加魔法的函数：

    void hv_magic(HV *hv, GV *gv, int how);

This simply calls C<sv_magic> and coerces the C<gv> argument into an C<SV>.

它仅仅是调用了sv_magic，并将gv参数强转为SV。

To remove the magic from an SV, call the function sv_unmagic:

为了移除SV的魔法，调用sv_unmagic：

    int sv_unmagic(SV *sv, int type);

The C<type> argument should be equal to the C<how> value when the C<SV>
was initially made magical.

type参数应该和SV被初始化为有魔法的时候的how参数相同。

However, note that C<sv_unmagic> removes all magic of a certain C<type> from the
C<SV>. If you want to remove only certain magic of a C<type> based on the magic
virtual table, use C<sv_unmagicext> instead:

但是，请注意sv_unmagic会移除SV中type类型所有的魔法。如果你想要基于魔法虚表只移除特定type的魔法，请使用sv_unmagicext以代替它。

    int sv_unmagicext(SV *sv, int type, MGVTBL *vtbl);

## 魔法虚表

=head2 Magic Virtual Tables

The C<mg_virtual> field in the C<MAGIC> structure is a pointer to an
C<MGVTBL>, which is a structure of function pointers and stands for
"Magic Virtual Table" to handle the various operations that might be
applied to that variable.

MAGIC结构的mg_virtual字段是一个指向MGVTBL结构的指针，它是一个函数指针结构，代表了"魔法虚表"以处理可能被用于那个变量的各种操作。

The C<MGVTBL> has five (or sometimes eight) pointers to the following
routine types:

MGVTBL有五个（或许有时是八个）指针指向如下函数类型：

    int  (*svt_get)(SV* sv, MAGIC* mg);
    int  (*svt_set)(SV* sv, MAGIC* mg);
    U32  (*svt_len)(SV* sv, MAGIC* mg);
    int  (*svt_clear)(SV* sv, MAGIC* mg);
    int  (*svt_free)(SV* sv, MAGIC* mg);

    int  (*svt_copy)(SV *sv, MAGIC* mg, SV *nsv,
                                          const char *name, I32 namlen);
    int  (*svt_dup)(MAGIC *mg, CLONE_PARAMS *param);
    int  (*svt_local)(SV *nsv, MAGIC *mg);


This MGVTBL structure is set at compile-time in F<perl.h> and there are
currently 32 types.  These different structures contain pointers to various
routines that perform additional actions depending on which function is
being called.

MGVTBL结构是在perl.h文件中编译期设置的，一共有32种类型。这些不同的结构包含了指向各种函数的指针，根据调用函数实现不同的功能。

   Function pointer    Action taken
   ----------------    ------------
   svt_get             Do something before the value of the SV is
                       retrieved.
   svt_set             Do something after the SV is assigned a value.
   svt_len             Report on the SV's length.
   svt_clear           Clear something the SV represents.
   svt_free            Free any extra storage associated with the SV.

   svt_copy            copy tied variable magic to a tied element
   svt_dup             duplicate a magic structure during thread cloning
   svt_local           copy magic to local value during 'local'

   函数指针             行为
   ----------------    ------------
   svt_get             在sv的值被获取到之前做一些事
   svt_set             在sv被赋值之后做一些事
   svt_len             报告sv的长度
   svt_clear           清除sv代表的内容
   svt_free            释放所有和sv关联的额外内存

   svt_copy            将绑定变量的魔法拷贝给绑定元素
   svt_dup             线程克隆时复制魔法结构
   svt_local           在'local'操作时赋值魔法给被local的值

For instance, the MGVTBL structure called C<vtbl_sv> (which corresponds
to an C<mg_type> of C<PERL_MAGIC_sv>) contains:

例如，MGVTBL结构之一vtbl_sv（对应PERL_MAGIC_sv这个mg_type）包含了：

    { magic_get, magic_set, magic_len, 0, 0 }

Thus, when an SV is determined to be magical and of type C<PERL_MAGIC_sv>,
if a get operation is being performed, the routine C<magic_get> is
called.  All the various routines for the various magical types begin
with C<magic_>.  NOTE: the magic routines are not considered part of
the Perl API, and may not be exported by the Perl library.

因此，当一个SV开始有类型为PERL_MAGIC_sv的魔法时，如果发生了一次取值操作，magic_get函数将会被调用。
所有魔法类型的函数都以magic_开头。注意：魔法函数并不是Perl API的一部分，可能不会被Perl库导出。

The last three slots are a recent addition, and for source code
compatibility they are only checked for if one of the three flags
MGf_COPY, MGf_DUP or MGf_LOCAL is set in mg_flags. This means that most
code can continue declaring a vtable as a 5-element value. These three are
currently used exclusively by the threading code, and are highly subject
to change.

最后三个槽是最近才添加的，为了保持兼容性，只有当mg_flags中MGF_COPY，MGF_DUP，MGF_LOCAL三个位被设置时才需要被检查。
这意味着大多数代码可以继续将vtable申明为一个有5个成员的值。这三个槽被用于线程相关的代码并且很可能会发生变化。

The current kinds of Magic Virtual Tables are:

当前的魔法虚表种类有：

=for comment
This table is generated by regen/mg_vtable.pl.  Any changes made here
will be lost.

=for mg_vtable.pl begin

 mg_type
 (old-style char and macro)   MGVTBL         Type of magic
 --------------------------   ------         -------------
 \0 PERL_MAGIC_sv             vtbl_sv        Special scalar variable
 #  PERL_MAGIC_arylen         vtbl_arylen    Array length ($#ary)
 %  PERL_MAGIC_rhash          (none)         extra data for restricted
                                             hashes
 &  PERL_MAGIC_proto          (none)         my sub prototype CV
 .  PERL_MAGIC_pos            vtbl_pos       pos() lvalue
 :  PERL_MAGIC_symtab         (none)         extra data for symbol
                                             tables
 <  PERL_MAGIC_backref        vtbl_backref   for weak ref data
 @  PERL_MAGIC_arylen_p       (none)         to move arylen out of XPVAV
 B  PERL_MAGIC_bm             vtbl_regexp    Boyer-Moore 
                                             (fast string search)
 c  PERL_MAGIC_overload_table vtbl_ovrld     Holds overload table 
                                             (AMT) on stash
 D  PERL_MAGIC_regdata        vtbl_regdata   Regex match position data 
                                             (@+ and @- vars)
 d  PERL_MAGIC_regdatum       vtbl_regdatum  Regex match position data
                                             element
 E  PERL_MAGIC_env            vtbl_env       %ENV hash
 e  PERL_MAGIC_envelem        vtbl_envelem   %ENV hash element
 f  PERL_MAGIC_fm             vtbl_regexp    Formline 
                                             ('compiled' format)
 g  PERL_MAGIC_regex_global   vtbl_mglob     m//g target
 H  PERL_MAGIC_hints          vtbl_hints     %^H hash
 h  PERL_MAGIC_hintselem      vtbl_hintselem %^H hash element
 I  PERL_MAGIC_isa            vtbl_isa       @ISA array
 i  PERL_MAGIC_isaelem        vtbl_isaelem   @ISA array element
 k  PERL_MAGIC_nkeys          vtbl_nkeys     scalar(keys()) lvalue
 L  PERL_MAGIC_dbfile         (none)         Debugger %_<filename
 l  PERL_MAGIC_dbline         vtbl_dbline    Debugger %_<filename
                                             element
 N  PERL_MAGIC_shared         (none)         Shared between threads
 n  PERL_MAGIC_shared_scalar  (none)         Shared between threads
 o  PERL_MAGIC_collxfrm       vtbl_collxfrm  Locale transformation
 P  PERL_MAGIC_tied           vtbl_pack      Tied array or hash
 p  PERL_MAGIC_tiedelem       vtbl_packelem  Tied array or hash element
 q  PERL_MAGIC_tiedscalar     vtbl_packelem  Tied scalar or handle
 r  PERL_MAGIC_qr             vtbl_regexp    precompiled qr// regex
 S  PERL_MAGIC_sig            (none)         %SIG hash
 s  PERL_MAGIC_sigelem        vtbl_sigelem   %SIG hash element
 t  PERL_MAGIC_taint          vtbl_taint     Taintedness
 U  PERL_MAGIC_uvar           vtbl_uvar      Available for use by
                                             extensions
 u  PERL_MAGIC_uvar_elem      (none)         Reserved for use by
                                             extensions
 V  PERL_MAGIC_vstring        (none)         SV was vstring literal
 v  PERL_MAGIC_vec            vtbl_vec       vec() lvalue
 w  PERL_MAGIC_utf8           vtbl_utf8      Cached UTF-8 information
 x  PERL_MAGIC_substr         vtbl_substr    substr() lvalue
 y  PERL_MAGIC_defelem        vtbl_defelem   Shadow "foreach" iterator
                                             variable / smart parameter
                                             vivification
 ]  PERL_MAGIC_checkcall      vtbl_checkcall inlining/mutation of call
                                             to this CV
 ~  PERL_MAGIC_ext            (none)         Available for use by
                                             extensions

=for mg_vtable.pl end

When an uppercase and lowercase letter both exist in the table, then the
uppercase letter is typically used to represent some kind of composite type
(a list or a hash), and the lowercase letter is used to represent an element
of that composite type. Some internals code makes use of this case
relationship.  However, 'v' and 'V' (vec and v-string) are in no way related.

当一个字母的大小写形式同时出现在表中，那么大写字母一般代表类型（一个列表或一个哈希表），小写字母一般代表那种类型的元素。
一些内部代码使用了这种字母类型的关系。然而，'v'和'V'（vec和v-string）是没有关系的。

The C<PERL_MAGIC_ext> and C<PERL_MAGIC_uvar> magic types are defined
specifically for use by extensions and will not be used by perl itself.
Extensions can use C<PERL_MAGIC_ext> magic to 'attach' private information
to variables (typically objects).  This is especially useful because
there is no way for normal perl code to corrupt this private information
(unlike using extra elements of a hash object).

PERL_MAGIC_ext和PERL_MAGIC_uvar魔法类型被定义用于拓展而不是被Perl自身使用。拓展可以使用PERL_MAGIC_ext魔法来讲私有信息'附着'到变量上（通常是对象）。
这中用法非常有用，因为perl代码没有办法破坏这些私有信息（与使用哈希表的额外元素不同）。

Similarly, C<PERL_MAGIC_uvar> magic can be used much like tie() to call a
C function any time a scalar's value is used or changed.  The C<MAGIC>'s
C<mg_ptr> field points to a C<ufuncs> structure:

与之类似，PERL_MAGIC_uvar魔法可以被用于像tie()一样，当一个scalar的值被使用或改变的时候调用一个C函数。MAGIC结构的mg_ptr字段指向一个ufuncs结构。

    struct ufuncs {
        I32 (*uf_val)(pTHX_ IV, SV*);
        I32 (*uf_set)(pTHX_ IV, SV*);
        IV uf_index;
    };

When the SV is read from or written to, the C<uf_val> or C<uf_set>
function will be called with C<uf_index> as the first arg and a pointer to
the SV as the second.  A simple example of how to add C<PERL_MAGIC_uvar>
magic is shown below.  Note that the ufuncs structure is copied by
sv_magic, so you can safely allocate it on the stack.

当SV被读取或写入的时候，将会调用uf_var或uf_set函数，第一个参数是uf_index，第二个参数是一个指向SV的指针。
一个添加PERL_MAGIC_uvar的简单示例如下所示。注意ufuncs结构被sv_magic拷贝，因此你可以安全得将其分配到栈上。

    void
    Umagic(sv)
        SV *sv;
    PREINIT:
        struct ufuncs uf;
    CODE:
        uf.uf_val   = &my_get_fn;
        uf.uf_set   = &my_set_fn;
        uf.uf_index = 0;
        sv_magic(sv, 0, PERL_MAGIC_uvar, (char*)&uf, sizeof(uf));

Attaching C<PERL_MAGIC_uvar> to arrays is permissible but has no effect.

将PERL_MAGIC_uvar附着到数组上也是允许的，但是没有任何作用。

For hashes there is a specialized hook that gives control over hash
keys (but not values).  This hook calls C<PERL_MAGIC_uvar> 'get' magic
if the "set" function in the C<ufuncs> structure is NULL.  The hook
is activated whenever the hash is accessed with a key specified as
an C<SV> through the functions C<hv_store_ent>, C<hv_fetch_ent>,
C<hv_delete_ent>, and C<hv_exists_ent>.  Accessing the key as a string
through the functions without the C<..._ent> suffix circumvents the
hook.  See L<Hash::Util::FieldHash/GUTS> for a detailed description.

对于哈希表，有一个特殊的手段可以控制哈希表的键。这个技巧调用PERL_MAGIC_uvar的'get'魔法（当ufuncs结构中的"set"函数为NULL的时候）。
当哈希表通过hv_store_ent，hv_fetch_ent，hv_delete_ent和hv_exists_ent几个函数以SV为键访问的时候这个特性将被激活。
当使用不带ent的函数以字符串为键访问的时候则不会激活。请看Hash::Util::FieldHash/GUTS以获得更多细节描述。

Note that because multiple extensions may be using C<PERL_MAGIC_ext>
or C<PERL_MAGIC_uvar> magic, it is important for extensions to take
extra care to avoid conflict.  Typically only using the magic on
objects blessed into the same class as the extension is sufficient.
For C<PERL_MAGIC_ext> magic, it is usually a good idea to define an
C<MGVTBL>, even if all its fields will be C<0>, so that individual
C<MAGIC> pointers can be identified as a particular kind of magic
using their magic virtual table. C<mg_findext> provides an easy way
to do that:

注意因为可能有多个拓展同时使用到PERL_MAGIC_ext和PERL_MAGIC_uvar魔法，避免冲突非常重要。
一般而言只对用被bessed为同一个类的对象使用魔法作为拓展就足够了。对于PERL_MAGIC_ext魔法，最好还是定义一个MGVTBL，
即使它所有的域都是0，这样的话特定的MAGIC指针能通过它们的特殊的魔法虚表标识出来。mg_findext为此提供了一个简单的方式：

    STATIC MGVTBL my_vtbl = { 0, 0, 0, 0, 0, 0, 0, 0 };

    MAGIC *mg;
    if ((mg = mg_findext(sv, PERL_MAGIC_ext, &my_vtbl))) {
        /* this is really ours, not another module's PERL_MAGIC_ext */
        my_priv_data_t *priv = (my_priv_data_t *)mg->mg_ptr;
        ...
    }

Also note that the C<sv_set*()> and C<sv_cat*()> functions described
earlier do B<not> invoke 'set' magic on their targets.  This must
be done by the user either by calling the C<SvSETMAGIC()> macro after
calling these functions, or by using one of the C<sv_set*_mg()> or
C<sv_cat*_mg()> functions.  Similarly, generic C code must call the
C<SvGETMAGIC()> macro to invoke any 'get' magic if they use an SV
obtained from external sources in functions that don't handle magic.
See L<perlapi> for a description of these functions.
For example, calls to the C<sv_cat*()> functions typically need to be
followed by C<SvSETMAGIC()>, but they don't need a prior C<SvGETMAGIC()>
since their implementation handles 'get' magic.

同样注意之前描述的sv_set*()和sv_cat*()函数不会引发它们目标上的'set'魔法。必须要在使用完这些函数后调用SVSETMAGIC()宏，
或者调用sv_set*_mg()湖sv_cat*_mg()函数才能做到。与此类似，C代码在从一些不处理魔法的函数中获取SV时必须要调用SvGETMAGIC()宏以获得'get'魔法。
请参考perapi了解这些函数的信息。
例如，调用sv_cat*()函数一般需要在后面接着调用SvSETMAGIC()，但是他们不需要在之前调用SvGETMAGIC()，因为他们的实现已经处理好'get'魔法了。

=head2 Finding Magic

    MAGIC *mg_find(SV *sv, int type); /* Finds the magic pointer of that
                                       * type */

This routine returns a pointer to a C<MAGIC> structure stored in the SV.
If the SV does not have that magical feature, C<NULL> is returned. If the
SV has multiple instances of that magical feature, the first one will be
returned. C<mg_findext> can be used to find a C<MAGIC> structure of an SV
based on both its magic type and its magic virtual table:

这个方法返回一个SV中指向MAGIC结构的指针。如果SV没有哪个魔法特性则返回NULL。如果SV有这个魔法特性的多个实例将会返回第一个。
mg_findext被用于通过魔法类型和魔法虚表找到SV的MAGIC结构。

    MAGIC *mg_findext(SV *sv, int type, MGVTBL *vtbl);

Also, if the SV passed to C<mg_find> or C<mg_findext> is not of type
SVt_PVMG, Perl may core dump.

同样，如果传入mg_find和mg_findext的SV不输入SVt_PVMG的话，Perl可能会发生内核转储。

    int mg_copy(SV* sv, SV* nsv, const char* key, STRLEN klen);

This routine checks to see what types of magic C<sv> has.  If the mg_type
field is an uppercase letter, then the mg_obj is copied to C<nsv>, but
the mg_type field is changed to be the lowercase letter.

这个函数检查sv的魔法类型。如果mg_type域是大写字符，那么mg_obj被拷贝的nsv，且mg_type域会被转为小写字符。

## 理解Tied哈希表和数组的魔法

=head2 Understanding the Magic of Tied Hashes and Arrays


Tied hashes and arrays are magical beasts of the C<PERL_MAGIC_tied>
magic type.

Tied哈希表和数组属于PERL_MAGIC_tied魔法类型。

WARNING: As of the 5.004 release, proper usage of the array and hash
access functions requires understanding a few caveats.  Some
of these caveats are actually considered bugs in the API, to be fixed
in later releases, and are bracketed with [MAYCHANGE] below. If
you find yourself actually applying such information in this section, be
aware that the behavior may change in the future, umm, without warning.

警告：在5.004版本，你需要理解一些注意事项才能正确得使用数组和哈希表的访问函数。一些注意事项其实是API的BUGS，以在后续版本修复，它们在下面会被用[可能改变]括起来。
如果你发现你使用了这部分的信息，请留心它们的行为在将来可能会变化。

The perl tie function associates a variable with an object that implements
the various GET, SET, etc methods.  To perform the equivalent of the perl
tie function from an XSUB, you must mimic this behaviour.  The code below
carries out the necessary steps - firstly it creates a new hash, and then
creates a second hash which it blesses into the class which will implement
the tie methods. Lastly it ties the two hashes together, and returns a
reference to the new tied hash.  Note that the code below does NOT call the
TIEHASH method in the MyTie class -
see L<Calling Perl Routines from within C Programs> for details on how
to do this.

perl tie函数将一个变量关联到一个实现了各种类似GET,SET等方法的对象。为了在XSUB函数中实现与perl tie函数一样的效果，你必须模仿这种行为。
下列代码包含了需要的步骤--首先创建一个新的哈希表，然后创建另一个哈希表并将其bless到一个实现了tie方法的类上面。最后将两个哈希表tie到一起，并返回一个新的tied哈希表。
注意下面的代码没有调用MyTie类中的TIEHASH方法--看 '在C程序中调用Perl函数' 一节以获取更多细节。 

    SV*
    mytie()
    PREINIT:
        HV *hash;
        HV *stash;
        SV *tie;
    CODE:
        hash = newHV();
        tie = newRV_noinc((SV*)newHV());
        stash = gv_stashpv("MyTie", GV_ADD);
        sv_bless(tie, stash);
        hv_magic(hash, (GV*)tie, PERL_MAGIC_tied);
        RETVAL = newRV_noinc(hash);
    OUTPUT:
        RETVAL

The C<av_store> function, when given a tied array argument, merely
copies the magic of the array onto the value to be "stored", using
C<mg_copy>.  It may also return NULL, indicating that the value did not
actually need to be stored in the array.  [MAYCHANGE] After a call to
C<av_store> on a tied array, the caller will usually need to call
C<mg_set(val)> to actually invoke the perl level "STORE" method on the
TIEARRAY object.  If C<av_store> did return NULL, a call to
C<SvREFCNT_dec(val)> will also be usually necessary to avoid a memory
leak. [/MAYCHANGE]

当传给av_store函数一个tied数组参数时，仅仅会使用mg_copy将数组的魔法拷贝到需要存储的值中。
它同样可能返回NULL，表明这个值实际上不需要被储存在数组中。[可能改变]
在对一个tired数组使用av_store后，调用者需要继续调用mg_set(val)以触发对tied数组对象perl级别的STORE方法。
如果av_store返回NULL，则需要调用SvREFCNG_dec(val)以避免内存泄漏。

The previous paragraph is applicable verbatim to tied hash access using the
C<hv_store> and C<hv_store_ent> functions as well.

之前的段落同样适用于tied使用hv_store和hv_store_ent访问哈希表。

C<av_fetch> and the corresponding hash functions C<hv_fetch> and
C<hv_fetch_ent> actually return an undefined mortal value whose magic
has been initialized using C<mg_copy>.  Note the value so returned does not
need to be deallocated, as it is already mortal.  [MAYCHANGE] But you will
need to call C<mg_get()> on the returned value in order to actually invoke
the perl level "FETCH" method on the underlying TIE object.  Similarly,
you may also call C<mg_set()> on the return value after possibly assigning
a suitable value to it using C<sv_setsv>,  which will invoke the "STORE"
method on the TIE object. [/MAYCHANGE]

av_fetch和对应的hv_fetch和hv_fetch_ent实际上返回的是一个用mg_copy初始化的未定义的将死值。
注意这个返回值不需要主动释放，因为它已经是将死的了。
[可能改变]
但是你你需要对这些返回值调用mg_get()以触发对tied对象perl级别的STORE方法。类似，在你对这些返回值使用sv_setsv赋值后可能需要调用mg_set()以出发STORE操作。
[/可能改变]

[MAYCHANGE]
In other words, the array or hash fetch/store functions don't really
fetch and store actual values in the case of tied arrays and hashes.  They
merely call C<mg_copy> to attach magic to the values that were meant to be
"stored" or "fetched".  Later calls to C<mg_get> and C<mg_set> actually
do the job of invoking the TIE methods on the underlying objects.  Thus
the magic mechanism currently implements a kind of lazy access to arrays
and hashes.

Currently (as of perl version 5.004), use of the hash and array access
functions requires the user to be aware of whether they are operating on
"normal" hashes and arrays, or on their tied variants.  The API may be
changed to provide more transparent access to both tied and normal data
types in future versions.
[/MAYCHANGE]

[可能改版]
从另一方面来看，对于tied数组和哈希表来说，fetch/store函数并不是真的获取或存储实际值。
他们仅仅是调用mg_copy来对处理的值施魔法。后续的mg_get和mg_set才是真正调用了TIE方法。
应此这个魔法机制目前实现的是一种对数组和哈希的延时的操作。

当前（对于perl 5.004版本来说），使用哈希表和数组访问方法需要用户清楚他们是在操作‘普通的’还是tied对象。
将来API可能会变化以提供更加透明的访问方式。
[/可能改版]

You would do well to understand that the TIEARRAY and TIEHASH interfaces
are mere sugar to invoke some perl method calls while using the uniform hash
and array syntax.  The use of this sugar imposes some overhead (typically
about two to four extra opcodes per FETCH/STORE operation, in addition to
the creation of all the mortal variables required to invoke the methods).
This overhead will be comparatively small if the TIE methods are themselves
substantial, but if they are only a few statements long, the overhead
will not be insignificant.

你肯定轻松理解到TIEARRAY和TIEHASH接口不过是使用哈希表和数组操作语法来调用一些perl函数的语法糖而已。
使用这些语法糖会导致某些性能损耗。但是如果你的TIE方法非常庞大的话这些损耗是可以忽略不计的。但是如果方法只有几条语句那么长的话，新能损耗就不可以被忽略了。

## 本地化改变

=head2 Localizing changes

Perl has a very handy construction

Perl有中非常易用的构造方式：

  {
    local $var = 2;
    ...
  }

This construction is I<approximately> equivalent to

这种方式大概类似于以下代码:

  {
    my $oldvar = $var;
    $var = 2;
    ...
    $var = $oldvar;
  }

The biggest difference is that the first construction would
reinstate the initial value of $var, irrespective of how control exits
the block: C<goto>, C<return>, C<die>/C<eval>, etc. It is a little bit
more efficient as well.

最大的不同是第一种构造将会$var的值将会复原，无论你以什么方式退出代码块。这样会更加高效。

There is a way to achieve a similar task from C via Perl API: create a
I<pseudo-block>, and arrange for some changes to be automatically
undone at the end of it, either explicit, or via a non-local exit (via
die()). A I<block>-like construct is created by a pair of
C<ENTER>/C<LEAVE> macros (see L<perlcall/"Returning a Scalar">).
Such a construct may be created specially for some important localized
task, or an existing one (like boundaries of enclosing Perl
subroutine/block, or an existing pair for freeing TMPs) may be
used. (In the second case the overhead of additional localization must
be almost negligible.) Note that any XSUB is automatically enclosed in
an C<ENTER>/C<LEAVE> pair.

可以在C语言中通过调用Perl API实现类似的功能：创建一个伪代码块，然后进行一些变化并在结尾处自动恢复，无论是主动的还是通过一次非正常退出（通过die函数）。
一个类似于代码块的构建是通过ENTER/LEAVE这一对宏实现的（参见perlcall/"返回一个scalar一节"）。像这样的构造也许会为一些特殊的需要本地化的任务热创建，
亦或本身就存在（比如Perl函数和代码块，or an existing pair for freeing TMPs）。（在第二种情况下不需要多余的本地化。）注意任何XSUB函数都被自动包含在一对ENTER/LEAVE对中。

Inside such a I<pseudo-block> the following service is available:

在这样一个伪代码块中一下服务是可用的：

=over 4

=item C<SAVEINT(int i)>

=item C<SAVEIV(IV i)>

=item C<SAVEI32(I32 i)>

=item C<SAVELONG(long i)>

These macros arrange things to restore the value of integer variable
C<i> at the end of enclosing I<pseudo-block>.

这些宏会将整数i的值在代码块结束后恢复。

=item C<SAVESPTR(s)>

=item C<SAVEPPTR(p)>

These macros arrange things to restore the value of pointers C<s> and
C<p>. C<s> must be a pointer of a type which survives conversion to
C<SV*> and back, C<p> should be able to survive conversion to C<char*>
and back.

这些宏会将指针s和p的值恢复。s必须满足可以被强转为SV*并复原，p必须满足可以被强转为char*并复原。

=item C<SAVEFREESV(SV *sv)>

The refcount of C<sv> would be decremented at the end of
I<pseudo-block>.  This is similar to C<sv_2mortal> in that it is also a
mechanism for doing a delayed C<SvREFCNT_dec>.  However, while C<sv_2mortal>
extends the lifetime of C<sv> until the beginning of the next statement,
C<SAVEFREESV> extends it until the end of the enclosing scope.  These
lifetimes can be wildly different.

sv的引用计数将在代码框的几位减一。这和sv_2mortal类似都是SvREFCNT_dec的延时操作。然而，sv_2mortal将sv的生命延长到下一条语句的开始，SAVEFREESV则延长到代码块的结束。
这些生命周期可能完全不同。

Also compare C<SAVEMORTALIZESV>.

=item C<SAVEMORTALIZESV(SV *sv)>

Just like C<SAVEFREESV>, but mortalizes C<sv> at the end of the current
scope instead of decrementing its reference count.  This usually has the
effect of keeping C<sv> alive until the statement that called the currently
live scope has finished executing.

和SAVEFREESV一样，但是在代码块的结束处将sv设置为将死的而不减少它的引用计数。这通常会使得sv生存到调用这个代码块的语句运行结束。

=item C<SAVEFREEOP(OP *op)>

The C<OP *> is op_free()ed at the end of I<pseudo-block>.

OP* 将在代码块末尾处被op_free()。

=item C<SAVEFREEPV(p)>

The chunk of memory which is pointed to by C<p> is Safefree()ed at the
end of I<pseudo-block>.

p指针指向的内存块将在代码块末尾处被Safefree()。

=item C<SAVECLEARSV(SV *sv)>

Clears a slot in the current scratchpad which corresponds to C<sv> at
the end of I<pseudo-block>.

在代码块结尾处清除表示一个sv的暂存器中的槽。

=item C<SAVEDELETE(HV *hv, char *key, I32 length)>

The key C<key> of C<hv> is deleted at the end of I<pseudo-block>. The
string pointed to by C<key> is Safefree()ed.  If one has a I<key> in
short-lived storage, the corresponding string may be reallocated like
this:

hv的键key在代码块的结尾处删除。key指向的字符串被safefree()掉。如果key被保存在短期储存空间中，对应的字符串可以这样重分配。

  SAVEDELETE(PL_defstash, savepv(tmpbuf), strlen(tmpbuf));

=item C<SAVEDESTRUCTOR(DESTRUCTORFUNC_NOCONTEXT_t f, void *p)>

At the end of I<pseudo-block> the function C<f> is called with the
only argument C<p>.

在代码块结尾，将调用f并传入参数p。

=item C<SAVEDESTRUCTOR_X(DESTRUCTORFUNC_t f, void *p)>

At the end of I<pseudo-block> the function C<f> is called with the
implicit context argument (if any), and C<p>.

在代码块结尾，将调用f，隐式传入上下文变量（如果有的话）和p。

=item C<SAVESTACK_POS()>

The current offset on the Perl internal stack (cf. C<SP>) is restored
at the end of I<pseudo-block>.

在代码块结尾，Perl内部栈偏移（cf。 SP）将被恢复。

=back

The following API list contains functions, thus one needs to
provide pointers to the modifiable data explicitly (either C pointers,
or Perlish C<GV *>s).  Where the above macros take C<int>, a similar
function takes C<int *>.

以下API列表包含函数，应此需要显式提供指向可变数据的指针（C指针或者Perlish的GV*）。比如如果以上的宏传入int的话，那么一个相似的函数就要传入int*。

=over 4

=item C<SV* save_scalar(GV *gv)>

Equivalent to Perl code C<local $gv>.

和Perl代码local $gv一样。

=item C<AV* save_ary(GV *gv)>

=item C<HV* save_hash(GV *gv)>

Similar to C<save_scalar>, but localize C<@gv> and C<%gv>.

和save_scalar一样，但是会本地化@gv和%gv。

=item C<void save_item(SV *item)>

Duplicates the current value of C<SV>, on the exit from the current
C<ENTER>/C<LEAVE> I<pseudo-block> will restore the value of C<SV>
using the stored value. It doesn't handle magic. Use C<save_scalar> if
magic is affected.

=item C<void save_list(SV **sarg, I32 maxsarg)>

A variant of C<save_item> which takes multiple arguments via an array
C<sarg> of C<SV*> of length C<maxsarg>.

=item C<SV* save_svref(SV **sptr)>

Similar to C<save_scalar>, but will reinstate an C<SV *>.

=item C<void save_aptr(AV **aptr)>

=item C<void save_hptr(HV **hptr)>

Similar to C<save_svref>, but localize C<AV *> and C<HV *>.

=back

The C<Alias> module implements localization of the basic types within the
I<caller's scope>.  People who are interested in how to localize things in
the containing scope should take a look there too.

=head1 Subroutines

=head2 XSUBs and the Argument Stack

The XSUB mechanism is a simple way for Perl programs to access C subroutines.
An XSUB routine will have a stack that contains the arguments from the Perl
program, and a way to map from the Perl data structures to a C equivalent.

The stack arguments are accessible through the C<ST(n)> macro, which returns
the C<n>'th stack argument.  Argument 0 is the first argument passed in the
Perl subroutine call.  These arguments are C<SV*>, and can be used anywhere
an C<SV*> is used.

Most of the time, output from the C routine can be handled through use of
the RETVAL and OUTPUT directives.  However, there are some cases where the
argument stack is not already long enough to handle all the return values.
An example is the POSIX tzname() call, which takes no arguments, but returns
two, the local time zone's standard and summer time abbreviations.

To handle this situation, the PPCODE directive is used and the stack is
extended using the macro:

    EXTEND(SP, num);

where C<SP> is the macro that represents the local copy of the stack pointer,
and C<num> is the number of elements the stack should be extended by.

Now that there is room on the stack, values can be pushed on it using C<PUSHs>
macro. The pushed values will often need to be "mortal" (See
L</Reference Counts and Mortality>):

    PUSHs(sv_2mortal(newSViv(an_integer)))
    PUSHs(sv_2mortal(newSVuv(an_unsigned_integer)))
    PUSHs(sv_2mortal(newSVnv(a_double)))
    PUSHs(sv_2mortal(newSVpv("Some String",0)))
    /* Although the last example is better written as the more
     * efficient: */
    PUSHs(newSVpvs_flags("Some String", SVs_TEMP))

And now the Perl program calling C<tzname>, the two values will be assigned
as in:

    ($standard_abbrev, $summer_abbrev) = POSIX::tzname;

An alternate (and possibly simpler) method to pushing values on the stack is
to use the macro:

    XPUSHs(SV*)

This macro automatically adjusts the stack for you, if needed.  Thus, you
do not need to call C<EXTEND> to extend the stack.

Despite their suggestions in earlier versions of this document the macros
C<(X)PUSH[iunp]> are I<not> suited to XSUBs which return multiple results.
For that, either stick to the C<(X)PUSHs> macros shown above, or use the new
C<m(X)PUSH[iunp]> macros instead; see L</Putting a C value on Perl stack>.

For more information, consult L<perlxs> and L<perlxstut>.

=head2 Autoloading with XSUBs

If an AUTOLOAD routine is an XSUB, as with Perl subroutines, Perl puts the
fully-qualified name of the autoloaded subroutine in the $AUTOLOAD variable
of the XSUB's package.

But it also puts the same information in certain fields of the XSUB itself:

    HV *stash           = CvSTASH(cv);
    const char *subname = SvPVX(cv);
    STRLEN name_length  = SvCUR(cv); /* in bytes */
    U32 is_utf8         = SvUTF8(cv);

C<SvPVX(cv)> contains just the sub name itself, not including the package.
For an AUTOLOAD routine in UNIVERSAL or one of its superclasses,
C<CvSTASH(cv)> returns NULL during a method call on a nonexistent package.

B<Note>: Setting $AUTOLOAD stopped working in 5.6.1, which did not support
XS AUTOLOAD subs at all.  Perl 5.8.0 introduced the use of fields in the
XSUB itself.  Perl 5.16.0 restored the setting of $AUTOLOAD.  If you need
to support 5.8-5.14, use the XSUB's fields.

=head2 Calling Perl Routines from within C Programs

There are four routines that can be used to call a Perl subroutine from
within a C program.  These four are:

    I32  call_sv(SV*, I32);
    I32  call_pv(const char*, I32);
    I32  call_method(const char*, I32);
    I32  call_argv(const char*, I32, char**);

The routine most often used is C<call_sv>.  The C<SV*> argument
contains either the name of the Perl subroutine to be called, or a
reference to the subroutine.  The second argument consists of flags
that control the context in which the subroutine is called, whether
or not the subroutine is being passed arguments, how errors should be
trapped, and how to treat return values.

All four routines return the number of arguments that the subroutine returned
on the Perl stack.

These routines used to be called C<perl_call_sv>, etc., before Perl v5.6.0,
but those names are now deprecated; macros of the same name are provided for
compatibility.

When using any of these routines (except C<call_argv>), the programmer
must manipulate the Perl stack.  These include the following macros and
functions:

    dSP
    SP
    PUSHMARK()
    PUTBACK
    SPAGAIN
    ENTER
    SAVETMPS
    FREETMPS
    LEAVE
    XPUSH*()
    POP*()

For a detailed description of calling conventions from C to Perl,
consult L<perlcall>.

=head2 Memory Allocation

=head3 Allocation

All memory meant to be used with the Perl API functions should be manipulated
using the macros described in this section.  The macros provide the necessary
transparency between differences in the actual malloc implementation that is
used within perl.

It is suggested that you enable the version of malloc that is distributed
with Perl.  It keeps pools of various sizes of unallocated memory in
order to satisfy allocation requests more quickly.  However, on some
platforms, it may cause spurious malloc or free errors.

The following three macros are used to initially allocate memory :

    Newx(pointer, number, type);
    Newxc(pointer, number, type, cast);
    Newxz(pointer, number, type);

The first argument C<pointer> should be the name of a variable that will
point to the newly allocated memory.

The second and third arguments C<number> and C<type> specify how many of
the specified type of data structure should be allocated.  The argument
C<type> is passed to C<sizeof>.  The final argument to C<Newxc>, C<cast>,
should be used if the C<pointer> argument is different from the C<type>
argument.

Unlike the C<Newx> and C<Newxc> macros, the C<Newxz> macro calls C<memzero>
to zero out all the newly allocated memory.

=head3 Reallocation

    Renew(pointer, number, type);
    Renewc(pointer, number, type, cast);
    Safefree(pointer)

These three macros are used to change a memory buffer size or to free a
piece of memory no longer needed.  The arguments to C<Renew> and C<Renewc>
match those of C<New> and C<Newc> with the exception of not needing the
"magic cookie" argument.

=head3 Moving

    Move(source, dest, number, type);
    Copy(source, dest, number, type);
    Zero(dest, number, type);

These three macros are used to move, copy, or zero out previously allocated
memory.  The C<source> and C<dest> arguments point to the source and
destination starting points.  Perl will move, copy, or zero out C<number>
instances of the size of the C<type> data structure (using the C<sizeof>
function).

=head2 PerlIO

The most recent development releases of Perl have been experimenting with
removing Perl's dependency on the "normal" standard I/O suite and allowing
other stdio implementations to be used.  This involves creating a new
abstraction layer that then calls whichever implementation of stdio Perl
was compiled with.  All XSUBs should now use the functions in the PerlIO
abstraction layer and not make any assumptions about what kind of stdio
is being used.

For a complete description of the PerlIO abstraction, consult L<perlapio>.

=head2 Putting a C value on Perl stack

A lot of opcodes (this is an elementary operation in the internal perl
stack machine) put an SV* on the stack. However, as an optimization
the corresponding SV is (usually) not recreated each time. The opcodes
reuse specially assigned SVs (I<target>s) which are (as a corollary)
not constantly freed/created.

Each of the targets is created only once (but see
L<Scratchpads and recursion> below), and when an opcode needs to put
an integer, a double, or a string on stack, it just sets the
corresponding parts of its I<target> and puts the I<target> on stack.

The macro to put this target on stack is C<PUSHTARG>, and it is
directly used in some opcodes, as well as indirectly in zillions of
others, which use it via C<(X)PUSH[iunp]>.

Because the target is reused, you must be careful when pushing multiple
values on the stack. The following code will not do what you think:

    XPUSHi(10);
    XPUSHi(20);