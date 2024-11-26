
find_package(Qt6 REQUIRED COMPONENTS
    Core Gui Widgets Xml)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

set(SOURCES main.cpp
    mainwindow.cpp)

set(core_SOURCES
    core/Backend.cpp 
    core/CanMessage.cpp 
    core/CanTrace.cpp 
    core/CanDbMessage.cpp 
    core/CanDb.cpp 
    core/CanDbNode.cpp 
    core/CanDbSignal.cpp 
    core/MeasurementSetup.cpp 
    core/MeasurementNetwork.cpp 
    core/MeasurementInterface.cpp 
    core/LogModel.cpp 
    core/ConfigurableWidget.cpp 
    core/Log.cpp
)

set(core_HEADERS
    core/Backend.h
    core/CanDb.h
    core/CanDbMessage.h
    core/CanDbNode.h
    core/CanDbSignal.h
    core/CanMessage.h
    core/CanTrace.h
    core/ConfigurableWidget.h
    core/Log.h
    core/LogModel.h
    core/MeasurementInterface.h
    core/MeasurementNetwork.h
    core/MeasurementSetup.h
    core/portable_endian.h
)

set(driver_SOURCES
    driver/CanDriver.cpp
    driver/CanInterface.cpp
    driver/CanListener.cpp
    driver/CanTiming.cpp
    driver/GenericCanSetupPage.cpp
    driver/SocketCanDriver/SocketCanDriver.cpp
    driver/SocketCanDriver/SocketCanInterface.cpp
)

set(driver_HEADERS
    driver/CanDriver.h
    driver/CanInterface.h
    driver/CanListener.h
    driver/CanTiming.h
    driver/GenericCanSetupPage.h
    driver/SocketCanDriver/SocketCanDriver.h
    driver/SocketCanDriver/SocketCanInterface.h
)

# if (WIN32)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(driver_SOURCES ${driver_SOURCES}
        driver/CandleApiDriver/CandleApiDriver.cpp
        driver/CandleApiDriver/CandleApiInterface.cpp
        driver/CandleApiDriver/CandleApiTiming.cpp
        driver/CandleApiDriver//api/candle.c
        driver/CandleApiDriver//api/candle_ctrl_req.c

        # TODO: PeakCan works with SocketCAN
        #   most of time, so it is _probably_ not needed
        #   in Linux, though a Linux driver exists.
        #   Someone may want to tinker this later.
        driver/PeakCanDriver/PeakCanDriver.cpp
        driver/PeakCanDriver/PeakCanInterface.cpp
    )

    set(driver_HEADERS ${driver_HEADERS}
        driver/CandleApiDriver/CandleApiDriver.h
        driver/CandleApiDriver/CandleApiInterface.h
        driver/CandleApiDriver/CandleApiTiming.h
        driver/CandleApiDriver/api/candle.h
        driver/CandleApiDriver/api/candle_ctrl_req.h
        driver/CandleApiDriver/api/candle_defs.h
        driver/CandleApiDriver/api/ch_9.h

        driver/PeakCanDriver/PeakCanDriver.h
        driver/PeakCanDriver/PeakCanInterface.h
    )

    set(EXTRA_LINK
        -lcandle_api -lSetupApi -lOle32 -lwinusb)

elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")

    # SocketCAN uses libnl3 that uses pkg-config.
    find_package(PkgConfig)
    pkg_search_module (LibNL
        REQUIRED libnl-route-3.0 libnl-route>=3)
        # REQUIRED libnl-route-3.0 libnl-3.0 libnl-3 libnl3 libnl>=3)

    set(EXTRA_LINK
        ${LibNL_LIBRARIES})
message("!!!! LibNL_LIBRARIES = ${LibNL_LIBRARIES} !!!!")
    find_path(LibNL_INCLUDE_DIR netlink/netlink.h
        /usr/include
        /usr/include/libnl3
        /usr/local/include
        /usr/local/include/libnl3
    )
    set(EXTRA_INC
        ${LibNL_INCLUDE_DIR})

endif (CMAKE_SYSTEM_NAME STREQUAL "Windows")

set(parser_dbc_SOURCES
    parser/dbc/DbcParser.cpp 
    parser/dbc/DbcTokens.cpp)
    
set(parser_dbc_HEADERS
    parser/dbc/DbcParser.h 
    parser/dbc/DbcTokens.h)

set(window_SOURCES
    window/CanStatusWindow/CanStatusWindow.cpp
    window/GraphWindow/GraphWindow.cpp
    window/LogWindow/LogWindow.cpp
    window/SetupDialog/SelectCanInterfacesDialog.cpp
    window/SetupDialog/SetupDialog.cpp
    window/SetupDialog/SetupDialogTreeItem.cpp
    window/SetupDialog/SetupDialogTreeModel.cpp
    window/TraceWindow/AggregatedTraceViewItem.cpp
    window/TraceWindow/AggregatedTraceViewModel.cpp
    window/TraceWindow/BaseTraceViewModel.cpp
    window/TraceWindow/LinearTraceViewModel.cpp
    window/TraceWindow/TraceWindow.cpp
) 

set(window_HEADERS
    window/CanStatusWindow/CanStatusWindow.h
    window/GraphWindow/GraphWindow.h
    window/LogWindow/LogWindow.h
    window/SetupDialog/SelectCanInterfacesDialog.h
    window/SetupDialog/SetupDialog.h
    window/SetupDialog/SetupDialogTreeItem.h
    window/SetupDialog/SetupDialogTreeModel.h
    window/TraceWindow/AggregatedTraceViewItem.h
    window/TraceWindow/AggregatedTraceViewModel.h
    window/TraceWindow/BaseTraceViewModel.h
    window/TraceWindow/LinearTraceViewModel.h
    window/TraceWindow/TraceViewTypes.h
    window/TraceWindow/TraceWindow.h    
)

set ( MOC_HEADERS
..h
)

set ( UIS
    driver/GenericCanSetupPage.ui
    mainwindow.ui
    window/CanStatusWindow/CanStatusWindow.ui
    window/GraphWindow/GraphWindow.ui
    window/LogWindow/LogWindow.ui
    window/SetupDialog/SelectCanInterfacesDialog.ui
    window/SetupDialog/SetupDialog.ui
    window/TraceWindow/TraceWindow.ui
)

# qt6_add_resources(RESOURCES cangaroo.qrc)
# set ( RESOURCES ..qrc)
# QT5_WRAP_UI( UI_HEADERS ${UIS} )

qt_add_executable( cangaroo
    ${SOURCES} ${core_SOURCES} ${driver_SOURCES}
    ${parser_dbc_SOURCES} ${window_SOURCES}
    ${UIS}
    ${RESOURCES}
)

target_include_directories(cangaroo
    PRIVATE 
    # ${PROJECT_SOURCE_DIR}
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>

    ${EXTRA_INC}
)

# qt6_use_modules
target_link_libraries(cangaroo PRIVATE 
    Qt::Core Qt::Gui Qt::Widgets Qt::Xml
    ${EXTRA_LINK})
