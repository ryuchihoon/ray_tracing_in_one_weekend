cmake_minimum_required(VERSION 3.0.0)

project(inOneWeekend VERSION 0.0.1 LANGUAGES CXX)

add_executable(inOneWeekend
	src/vec3.h
	src/color.h
	src/main.cc
)

set_target_properties(inOneWeekend
PROPERTIES
    CXX_STANDARD 17
)
