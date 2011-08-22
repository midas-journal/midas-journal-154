CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

INCLUDE(${CMAKE_SOURCE_DIR}/IJMacros.txt)

#Change PROJECT_NAME to the name of your project
PROJECT(DemonsRegistration)

#The following lines are required to use Dart
ENABLE_TESTING()
INCLUDE(Dart)

#Declare any external dependencies that your project may have here.
#examples include: ITK, VTK, JPEG, PNG, OpenGL, ZLIB, Perl, Java
#If you're not sure what name to use, look in the Modules directory of your
#cmake install and check that a file named Find(Package).cmake exists
SET(Required_Packages
  ITK
  )

#this foreach loads all of the packages that you specified as required.
#It shouldn't need to be modified.
FOREACH(Package ${Required_Packages})
  LOADPACKAGE(${Package})
ENDFOREACH(Package)

#Some packages options are required for this project. Check them now
SET(_ITKVersionString "${ITK_VERSION_MAJOR}.${ITK_VERSION_MINOR}.${ITK_VERSION_PATCH}" )
MATH(EXPR _ITKVersionNum "${ITK_VERSION_MAJOR}*100*100 + ${ITK_VERSION_MINOR}*100 + ${ITK_VERSION_PATCH}")
IF( _ITKVersionNum LESS 30800 )
  MESSAGE(SEND_ERROR "The ITK version you want to use (${_ITKVersionString}) is not supported by this project. Please use a more recent version of ITK. The minimum required version is 3.8.0. The recommended version is 3.14.0.")
ENDIF( _ITKVersionNum LESS 30800 )
IF(NOT ITK_USE_REVIEW)
  MESSAGE(SEND_ERROR "ITK_USE_REVIEW has not been set to on when compiling ITK. This option is mandatory for this project to build. Please recompile your ITK installation.")
ENDIF(NOT ITK_USE_REVIEW)

#Set any libraries that your project depends on.
#examples: ITKCommon, VTKRendering, etc
SET(Libraries
  ITKCommon
  ITKIO
  ITKStatistics
)

#the following block of code is an example of how to build an executable in
#cmake.  Unmodified, it will add an executable called "MyExe" to the project.
#MyExe will be built using the files MyClass.h and MyClass.cxx, and it will
#be linked to all the libraries you specified above. 
#You can build more than one executable per project
SET(CurrentExe "ImageCompare")
ADD_EXECUTABLE(${CurrentExe} ImageCompare.cxx) 
TARGET_LINK_LIBRARIES(${CurrentExe} ${Libraries})

SET(CurrentExe "DemonsRegistration")
ADD_EXECUTABLE(${CurrentExe} DemonsRegistration.cxx)
TARGET_LINK_LIBRARIES(${CurrentExe} ${Libraries})

#the following line is an example of how to add a test to your project.
#Testname is the title for this particular test.  ExecutableToRun is the
#program which will be running this test.  It can either be a part of this
#project or an external executable.  After that list any args that are needed
#for this test.  Include as many tests as you like.  If your project doesn't have
#any tests you can comment out or delete the following line.


SET(MY_EXE_DIR ${CMAKE_BINARY_DIR})
IF(RUNTIME_OUTPUT_DIRECTORY)
   SET(MY_EXE_DIR ${RUNTIME_OUTPUT_DIRECTORY})
ENDIF(RUNTIME_OUTPUT_DIRECTORY)

# Also do a unit test that checks whether the registration result is the same
# whether we use pixel spacing and image direction or not
ADD_TEST(RegisterImages ${MY_EXE_DIR}/DemonsRegistration
  -f ${CMAKE_SOURCE_DIR}/RatLungSlice1.mhd
  -m ${CMAKE_SOURCE_DIR}/RatLungSlice2.mhd
  -o ${PROJECT_BINARY_DIR}/output.mha
  -d -s 2 -g 1 -i 3x5 -e)

ADD_TEST(RegisterImagesS ${MY_EXE_DIR}/DemonsRegistration
  -f ${CMAKE_SOURCE_DIR}/RatLungSlice1s.mhd
  -m ${CMAKE_SOURCE_DIR}/RatLungSlice2s.mhd
  -o ${PROJECT_BINARY_DIR}/output-s.mha
  -d -s 2 -g 1 -i 3x5 -e)

ADD_TEST(CompareImages ${MY_EXE_DIR}/ImageCompare 
   ${PROJECT_BINARY_DIR}/output.mha ${PROJECT_BINARY_DIR}/output-s.mha)

#Once you're done modifying this template, you should rename it to "CMakeLists.txt"
