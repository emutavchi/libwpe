project('libwpe', 'cpp', 'c',
	meson_version: '>=0.49',
	default_options: [
		'b_ndebug=if-release',
		'c_std=c99',
		'cpp_eh=none',
		'cpp_std=c++11',
	],
	license: 'BSD-2-Clause',
	version: '1.7.1',
)

# This refers to the API level provided. This is modified only with major,
# breaking changes, which need modifications in programs using the library
# before they can be compiled again.
api_version = '1.0'

# Before making a release, the LT_VERSION string should be modified.
# The string is of the form [C, R, A].
# - If interfaces have been changed or added, but binary compatibility has
#   been preserved, change to [C+1, 0, A+1].
# - If binary compatibility has been broken (eg removed or changed interfaces)
#   change to [C+1, 0, 0]
# - If the interface is the same as the previous version, use [C, R+1, A].
soversion = [5, 1, 4]

# Split the *project* version into its components.
version_info = meson.project_version().split('.')
version_info = {
	'PROJECT_VERSION_MAJOR': version_info[0],
	'PROJECT_VERSION_MINOR': version_info[1],
	'PROJECT_VERSION_PATCH': version_info[2],
}

# Mangle [C, R, A] into an actual usable *soversion*.
soversion_major = soversion[0] - soversion[2]  # Current-Age
soversion_minor = soversion[2]  # Age
soversion_micro = soversion[1]  # Revision
soversion = '@0@.@1@.@2@'.format(soversion_major, soversion_minor, soversion_micro)

add_project_arguments('-DWPE_COMPILATION=1', language: ['c', 'cpp'])

# Switch to the 'cpp_rtti=false' default option when updating to Meson 0.53 or newer, see
# https://mesonbuild.com/FAQ.html#how-do-i-disable-exceptions-and-rtti-in-my-c-project
add_project_arguments(
	meson.get_compiler('cpp').get_supported_arguments(['-fno-rtti']),
	language: 'cpp'
)

default_backend = get_option('default-backend').strip()
if default_backend != ''
	add_project_arguments('-DWPE_BACKEND="@0@"'.format(default_backend), language: ['c', 'cpp'])
endif

dependencies = [
	dependency('xkbcommon'),
]

cc = meson.get_compiler('c')
if not cc.has_header('EGL/eglplatform.h')
	dependencies += dependency('egl')
endif

if not cc.has_function('dlopen')
	dependencies += cc.find_library('dl')
endif

libwpe = library('wpe-' + api_version,
	'src/input.c',
	'src/key-unicode.c',
	'src/loader.c',
	'src/pasteboard.c',
	'src/pasteboard-generic.cpp',
	'src/pasteboard-noop.cpp',
	'src/renderer-backend-egl.c',
	'src/renderer-host.c',
	'src/version.c',
	'src/view-backend.c',
	install: true,
	dependencies: dependencies,
	version: soversion,
	soversion: soversion_major,
	include_directories: 'include',
	gnu_symbol_visibility: 'hidden',
)

api_headers = [
	'include/wpe/export.h',
	'include/wpe/input.h',
	'include/wpe/keysyms.h',
	'include/wpe/loader.h',
	'include/wpe/pasteboard.h',
	'include/wpe/renderer-backend-egl.h',
	'include/wpe/renderer-host.h',
	'include/wpe/view-backend.h',
	'include/wpe/wpe-egl.h',
	'include/wpe/wpe.h',

	# Generated API headers.
	configure_file(
		input: 'include/wpe/version.h.cmake',
		output: 'version.h',
		configuration: version_info,
	),
	configure_file(
		input: 'include/wpe/version-deprecated.h.cmake',
		output: 'version-deprecated.h',
		configuration: version_info,
	),
]
install_headers(api_headers,
	subdir: join_paths('wpe-' + api_version, 'wpe'),
)

import('pkgconfig').generate(
	description: 'The wpe library',
	name: 'wpe-' + api_version,
	subdirs: 'wpe-' + api_version,
	libraries: libwpe,
	version: meson.project_version(),
)

if get_option('build-docs')
	hotdoc = import('hotdoc')
	assert(hotdoc.has_extensions('c-extension'),
		'The HotDoc C extension is required.'
	)
	libwpe_doc = hotdoc.generate_doc(meson.project_name(),
		project_version: api_version,
		dependencies: dependencies,
		index: 'docs/index.md',
		sitemap: 'docs/sitemap.txt',
		console: true,
		install: true,
		build_by_default: true,
		c_smart_index: true,
		c_sources: api_headers,
		c_include_directories: [
			include_directories('include'),
			meson.current_build_dir()
		],
		# The space here is relevant, see
		# https://github.com/mesonbuild/meson/issues/5800#issuecomment-552198354
		extra_c_flags: [' -DWPE_COMPILATION=1'],
	)
endif
