# External Project Patch

ExternalProject_Add经常重复Patch, 导致执行cmake --build执行不下去，例如下面的CMakeLists.txt里的命令:


```
ExternalProject_Add(etcd
    PREFIX ${thirdpartyDir}/etcd
    GIT_REPOSITORY https://gitee.com/mirrors/etcd
    GIT_TAG v3.4.0
    GIT_PROGRESS true
    GIT_REMOTE_UPDATE_STRATEGY REBASE
    SOURCE_DIR ${downloadDir}/etcd
    CONFIGURE_COMMAND "" BUILD_COMMAND ""
    INSTALL_COMMAND ""
    PATCH_COMMAND
        patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/etcdclient/expose-session-for-election.patch
)

重复执行cmake --build .

[ 50%] Built target golang
[ 56%] Performing update step for 'etcd'
[ 62%] Performing patch step for 'etcd'
patching file clientv3/concurrency/election.go
Reversed (or previously applied) patch detected!  Assume -R? [n]
```

方法之一是为patch command加入一个guard，只做一次patch:
```
ExternalProject_Add(etcd
    PREFIX ${thirdpartyDir}/etcd
    GIT_REPOSITORY https://gitee.com/mirrors/etcd
    GIT_TAG v3.4.0
    GIT_PROGRESS true
    GIT_REMOTE_UPDATE_STRATEGY REBASE
    SOURCE_DIR ${downloadDir}/etcd
    CONFIGURE_COMMAND "" BUILD_COMMAND ""
    INSTALL_COMMAND ""
    LIST_SEPARATOR @
    PATCH_COMMAND bash -c "cd <SOURCE_DIR>@ if ! [ -e i_am_patched ]@ then\
        patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/etcdclient/expose-session-for-election.patch@ touch i_am_patched@ fi"
)
```

以上假定补丁文件不变，如果补丁文件变化了，那只能想其他办法，最简单的办法是重新下载。
