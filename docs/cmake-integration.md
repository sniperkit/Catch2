<a id="top"></a>
# CMake integration

Because we use CMake to build Catch2, we also provide a couple of
integration points for our users.

1) Catch2 exports a (namespaced) CMake target
2) Catch2's repository contains CMake scripts for automatic registration
of `TEST_CASE`s in CTest

## CMake targets for linking


* As subdirectory

* Installed


## CMake scripts that register tests with CTest

* ParseAndAddCatchTests

* CatchAddTests



--------------------------------------
obsolete

In general we recommend "vendoring" Catch's single-include releases inside your own repository. If you do this, the following example shows a minimal CMake project:
```CMake
cmake_minimum_required(VERSION 3.0)

project(cmake_test)

# Prepare "Catch" library for other executables
set(CATCH_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/catch)
add_library(Catch INTERFACE)
target_include_directories(Catch INTERFACE ${CATCH_INCLUDE_DIR})

# Make test executable
set(TEST_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)
add_executable(tests ${TEST_SOURCES})
target_link_libraries(tests Catch)
```
Note that it assumes that the path to the Catch's header is `catch/catch.hpp` from the `CMakeLists.txt` file.


You can also use the following CMake snippet to automatically fetch the entire Catch repository from github and configure it as an external project:
```CMake
cmake_minimum_required(VERSION 2.8.8)
project(catch_builder CXX)
include(ExternalProject)
find_package(Git REQUIRED)

ExternalProject_Add(
    catch
    PREFIX ${CMAKE_BINARY_DIR}/catch
    GIT_REPOSITORY https://github.com/philsquared/Catch.git
    TIMEOUT 10
    UPDATE_COMMAND ${GIT_EXECUTABLE} pull
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    LOG_DOWNLOAD ON
   )

# Expose required variable (CATCH_INCLUDE_DIR) to parent scope
ExternalProject_Get_Property(catch source_dir)
set(CATCH_INCLUDE_DIR ${source_dir}/single_include CACHE INTERNAL "Path to include folder for Catch")
```

If you put it in, e.g., `${PROJECT_SRC_DIR}/${EXT_PROJECTS_DIR}/catch/`, you can use it in your project by adding the following to your root CMake file:

```CMake
# Includes Catch in the project:
add_subdirectory(${EXT_PROJECTS_DIR}/catch)
include_directories(${CATCH_INCLUDE_DIR} ${COMMON_INCLUDES})
enable_testing(true)  # Enables unit-testing.
```

The advantage of this approach is that you can always automatically update Catch to the latest release. The disadvantage is that it means bringing in lot more than you need.


### Automatic test registration
We provide 2 CMake scripts that can automatically register Catch-based
tests with CTest,
  * `contrib/ParseAndAddCatchTests.cmake`
  * `contrib/CatchAddTests.cmake`

The first is based on parsing the test implementation files, and attempts
to register all `TEST_CASE`s using their tags as labels. This means that
these:

```cpp
TEST_CASE("Test1", "[unit]") {
    int a = 1;
    int b = 2;
    REQUIRE(a == b);
}

TEST_CASE("Test2") {
    int a = 1;
    int b = 2;
    REQUIRE(a == b);
}

TEST_CASE("Test3", "[a][b][c]") {
    int a = 1;
    int b = 2;
    REQUIRE(a == b);
}
```
would be registered as 3 tests, `Test1`, `Test2` and `Test3`,
and 4 CTest labels would be created, `a`, `b`, `c` and `unit`.


The second is based on parsing the output of a Catch binary given
`--list-test-names-only`. This means that it deals with inactive
(e.g. commented-out) tests better, but requires CMake 3.10 for full
functionality.

### CodeCoverage module (GCOV, LCOV...)

If you are using GCOV tool to get testing coverage of your code, and are not sure how to integrate it with CMake and Catch, there should be an external example over at https://github.com/fkromer/catch_cmake_coverage

---

[Home](Readme.md#top)
