cmake_minimum_required(VERSION 4.00)

project(MyProject CXX)

find_package(fmt CONFIG REQUIRED)

add_executable(my_program main.cpp)

target_link_libraries(my_program PRIVATE fmt::fmt)
