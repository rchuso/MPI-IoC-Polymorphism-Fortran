MESSAGE( STATUS "    --> in 'apps'" )

ADD_EXECUTABLE( TestMpiIoc TestMpiIoc.f90 MsgTest.f90 )
TARGET_LINK_LIBRARIES( TestMpiIoc LibMpiStatic )
