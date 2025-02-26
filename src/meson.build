# SPDX-FileCopyrightText: 2021 Andrea Pappacoda
#
# SPDX-License-Identifier: AGPL-3.0-or-later

# If get_option('sysconfdir') returns an absolute path,
# get_option('prefix')/get_option('sysconfdir') simply returns the
# correct absolute sysconfdir, stripping away the prefix.
# This means that concatenating the prefix with the sysconfdir always
# yelds an absolute path that is correct, regardless of the fact that
# sysconfdir is absolute or not (e.g. /etc vs /usr/local/etc).
# Source: https://github.com/mesonbuild/meson/pull/9903#issuecomment-1071118017
# and: https://mesonbuild.com/Reference-manual_functions.html#join_paths

INIReader_dep = dependency(
	'INIReader',
	default_options: ['default_library=static', 'distro_install=false']
)

executable(
	'cloudflare-ddns',
	'main.cpp',
	dependencies: [
		cloudflare_ddns_dep,
		libcurl_dep,
		simdjson_dep,
		INIReader_dep,
		dependency('threads'),
		extra_deps
	],
	gnu_symbol_visibility: 'hidden',
	install: true,
	sources: configure_file(
		input: 'config_path.hpp.in',
		output: 'config_path.hpp',
		configuration: {'sysconfdir': get_option('prefix')/get_option('sysconfdir')}
	)
)

install_data(
	'config.ini',
	install_dir: get_option('sysconfdir')/'cloudflare-ddns',
	install_mode: ['rw-------', 'cloudflare-ddns', 'cloudflare-ddns']
)

# https://github.com/mesonbuild/meson/issues/1550
# https://github.com/mesonbuild/meson/pull/9342
ronn = find_program('ronn', required: get_option('man'))
if ronn.found()
	custom_target(
		'man',
		command: [ronn, '--roff', '--output-dir', meson.current_build_dir(), '@INPUT@'],
		input: 'cloudflare-ddns.1.ronn',
		output: 'cloudflare-ddns.1',
		install: true,
		install_dir: get_option('mandir')/'man1'
	)
endif

if host_machine.system() == 'linux'
	systemddir = 'lib'/'systemd'/'system'

	configure_file(
		input: 'systemd'/'cloudflare-ddns.service.in',
		output: 'cloudflare-ddns.service',
		configuration: {
			'bindir': get_option('prefix')/get_option('bindir'),
			'libdir': get_option('prefix')/get_option('libdir')
		},
		install: true,
		install_dir: systemddir
	)

	install_data(
		'systemd'/'cloudflare-ddns.timer',
		install_dir: systemddir
	)

	install_data(
		'sysusers.d'/'cloudflare-ddns.conf',
		install_dir: 'lib'/'sysusers.d'
	)
endif
