# unixODBC 2.3.7 on loongarch64

## 必须打 sqltypes.h 补丁

Doris 的 `thirdparty/patches/sqltypes.h.patch` 把 `TCHAR` 改名为 `TWCHAR`。否则在同时引入 ODBC 与 clucene 头文件的编译单元里会冲突：

    /usr/include/sqltypes.h:95: error: conflicting declaration ‘typedef char TCHAR’
    .../clucene-config.h:55: note: previous declaration as ‘typedef wchar_t TCHAR’

原因是 unixODBC 的 `sqltypes.h` 在未定义 `UNICODE` 时 `typedef char TCHAR`，而 clucene 用 `wchar_t`。

## 构建（GCC 16 需要降标准）

unixODBC 2.3.7 是老 K&R 风格代码，GCC 16 默认 C23 会报 `too many arguments to function ... expected 0`：

    cp ~/opt/tools/usr/share/automake-1.18/config.guess .   # 老 config.guess 同样不认 loongarch64
    cp ~/opt/tools/usr/share/automake-1.18/config.sub .
    ./configure --prefix=$PREFIX --enable-static --disable-shared --disable-gui --disable-drivers \
        CFLAGS="-std=gnu17 -O2 -Wno-implicit-function-declaration -Wno-int-conversion \
                -Wno-incompatible-pointer-types -Wno-implicit-int"
    make -j4 && make install

## 更新（2026-11-04）

打过 `sqltypes.h.patch` 的这份解决了 clucene 的 `TCHAR` 冲突，Exprs / Exec / Olap 全量编译通过。
链接行另需 `-lltdl`（unixODBC 引用），最终 0 undefined symbol。
