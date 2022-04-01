ADD_EXECUTABLE(main main.c)
INCLUDE_DIRECTORIES(/tmp/t3/usr/include/hello)
LINK_DIRECTORIES(/tmp/t3/usr/lib)
TARGET_LINK_LIBRARIES(main libhello.a)
