cmake_minimum_required(VERSION 2.8)

project(dnsmasq C)

execute_process(COMMAND bld/get-version ${CMAKE_CURRENT_SOURCE_DIR} 
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} 
		OUTPUT_VARIABLE FULL_VERSION)

set(VERSION_REGEXP "([0-9]+).([0-9]+)rc([0-9]+)-([0-9]+)-([a-z0-9]+)")

string(REGEX REPLACE "${VERSION_REGEXP}" "\\1;\\2;\\3;\\4;\\5" VERSION_LIST ${FULL_VERSION} )

list(GET VERSION_LIST 0 DNSMASQ_VERSION_MAJOR)
list(GET VERSION_LIST 1 DNSMASQ_VERSION_MINOR)
list(GET VERSION_LIST 2 DNSMASQ_VERSION_RCNO)
list(GET VERSION_LIST 3 DNSMASQ_VERSION_PATCH)
list(GET VERSION_LIST 4 DNSMASQ_VERSION_COMMIT)

add_definitions(-DVERSION="${DNSMASQ_VERSION_MAJOR}.${DNSMASQ_VERSION_MINOR}")

set(CMAKE_INSTALL_BINDIR ${CMAKE_INSTALL_PREFIX}/sbin)
set(CMAKE_INSTALL_SYSCONFDIR ${CMAKE_INSTALL_PREFIX}/etc)
set(CMAKE_INSTALL_MANDIR ${CMAKE_INSTALL_PREFIX}/share/man)
set(CMAKE_INSTALL_LOCALEDIR ${CMAKE_INSTALL_PREFIX}/share/locale)

set(CMAKE_C_FLAGS "-Wall -W")
set(CMAKE_C_FLAGS_DEBUG "-O0 -ggdb")
set(CMAKE_C_FLAGS_RELEASE "-O2")

#find_package(Dbus)
#find_package(idn)
#find_package(idn2)
#find_package(Conntrack)
#find_package(Lua)
#find_package(Nettle)
#find_package(Gmp)

set(CMAKE_C_STANDARD 99)

if (CMAKE_SNAP)
    add_definitions(-DRUNFILE=NULL)
endif (CMAKE_SNAP)

set(SOURCE_FILES
        src/arp.c
        src/auth.c
        src/blockdata.c
        src/bpf.c
        src/cache.c
        src/config.h
        src/conntrack.c
        src/crypto.c
        src/dbus.c
        src/dhcp-common.c
        src/dhcp-protocol.h
        src/dhcp.c
        src/dhcp6-protocol.h
        src/dhcp6.c
        src/dns-protocol.h
        src/dnsmasq.c
        src/dnsmasq.h
        src/dnssec.c
        src/domain.c
        src/edns0.c
        src/forward.c
        src/helper.c
        src/inotify.c
        src/ip6addr.h
        src/ipset.c
        src/lease.c
        src/log.c
        src/loop.c
        src/netlink.c
        src/network.c
        src/option.c
        src/outpacket.c
        src/poll.c
        src/radv-protocol.h
        src/radv.c
        src/rfc1035.c
        src/rfc2131.c
        src/rfc3315.c
        src/rrfilter.c
        src/slaac.c
        src/tables.c
        src/tftp.c
        src/util.c)

add_executable(dnsmasq ${SOURCE_FILES})

if (CMAKE_FULL_INSTALLATION)
    install(FILES man/dnsmasq.8
            DESTINATION ${CMAKE_INSTALL_MANDIR}/man8
            PERMISSIONS OWNER_READ OWNER_EXECUTE GROUP_READ WORLD_READ)
endif()

configure_file(dnsmasq.conf.in ${CMAKE_CURRENT_BINARY_DIR}/dnsmasq.conf @ONLY )
execute_process(COMMAND sed -i -e "s/#.*$//" -i -e "/^$/d" ${CMAKE_CURRENT_BINARY_DIR}/dnsmasq.conf
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		OUTPUT_FILE ${CMAKE_CURRENT_BINARY_DIR}/sed.log)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/dnsmasq.conf
        DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}
        PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)
install(TARGETS ${PROJECT_NAME}
        DESTINATION ${CMAKE_INSTALL_BINDIR}
        PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
