project(
	'wlroots',
	'c',
	version: '0.12.0',
	license: 'MIT',
	meson_version: '>=0.54.0',
	default_options: [
		'c_std=c11',
		'warning_level=2',
		'werror=false',
	],
)

# When doing a major or minor release, *always* increase soversion. This isn't
# necessary for bugfix releases. Increasing soversion is required because
# wlroots never guarantees ABI stability -- only API stability is guaranteed
# between minor releases.
soversion = '7d'

add_project_arguments([
	'-DWLR_USE_UNSTABLE',
], language: 'c')

cc = meson.get_compiler('c')

add_project_arguments(cc.get_supported_arguments([
	'-Wundef',
	'-Wlogical-op',
	'-Wmissing-include-dirs',
	'-Wold-style-definition',
	'-Wpointer-arith',
	'-Winit-self',
	'-Wstrict-prototypes',
	'-Wimplicit-fallthrough=2',
	'-Wendif-labels',
	'-Wstrict-aliasing=2',
	'-Woverflow',
	'-Wmissing-prototypes',

	'-Wno-missing-braces',
	'-Wno-missing-field-initializers',
	'-Wno-unused-parameter',
]), language: 'c')

add_project_link_arguments(
       [
               '-lhardware',
               '-lsync',
               '-lhybris-hwcomposerwindow',
               '-lhwc2',
               '-lhybris-common'
       ],
       language: 'c',
)


# Compute the relative path used by compiler invocations.
source_root = meson.current_source_dir().split('/')
build_root = meson.build_root().split('/')
relative_dir_parts = []
i = 0
in_prefix = true
foreach p : build_root
	if i >= source_root.length() or not in_prefix or p != source_root[i]
		in_prefix = false
		relative_dir_parts += '..'
	endif
	i += 1
endforeach
i = 0
in_prefix = true
foreach p : source_root
	if i >= build_root.length() or not in_prefix or build_root[i] != p
		in_prefix = false
		relative_dir_parts += p
	endif
	i += 1
endforeach
relative_dir = join_paths(relative_dir_parts) + '/'

# Strip relative path prefixes from the code if possible, otherwise hide them.
if cc.has_argument('-fmacro-prefix-map=/prefix/to/hide=')
	add_project_arguments(
		'-fmacro-prefix-map=@0@='.format(relative_dir),
		language: 'c',
	)
else
	add_project_arguments(
		'-DWLR_REL_SRC_DIR="@0@"'.format(relative_dir),
		language: 'c',
	)
endif

conf_data = configuration_data()
conf_data.set10('WLR_HAS_SYSTEMD', false)
conf_data.set10('WLR_HAS_ELOGIND', false)
conf_data.set10('WLR_HAS_LIBSEAT', false)
conf_data.set10('WLR_HAS_X11_BACKEND', false)
conf_data.set10('WLR_HAS_XWAYLAND', false)
conf_data.set10('WLR_HAS_XCB_ERRORS', false)
conf_data.set10('WLR_HAS_XCB_ICCCM', false)
conf_data.set10('WLR_HAS_EGLMESAEXT_H', false)
conf_data.set10('WLR_HAS_DROIDIAN_EXTENSIONS', false)

# Clang complains about some zeroed initializer lists (= {0}), even though they
# are valid
if cc.get_id() == 'clang'
	add_project_arguments('-Wno-missing-field-initializers', language: 'c')
	add_project_arguments('-Wno-missing-braces', language: 'c')
endif

wayland_server = dependency('wayland-server', version: '>=1.18')
wayland_client = dependency('wayland-client')
wayland_egl = dependency('wayland-egl')
wayland_protos = dependency('wayland-protocols', version: '>=1.21')
egl = dependency('egl')
glesv2 = dependency('glesv2')
drm = dependency('libdrm', version: '>=2.4.95')
gbm = dependency('gbm', version: '>=17.1.0')
libinput = dependency('libinput', version: '>=1.9.0')
xkbcommon = dependency('xkbcommon')
udev = dependency('libudev')
pixman = dependency('pixman-1')
math = cc.find_library('m')
rt = cc.find_library('rt')

if cc.has_header('EGL/eglmesaext.h', dependencies: egl)
	conf_data.set10('WLR_HAS_EGLMESAEXT_H', true)
endif

if get_option('with-droidian-extensions')
	conf_data.set10('WLR_HAS_DROIDIAN_EXTENSIONS', true)
endif

wlr_files = []
wlr_deps = [
	wayland_server,
	wayland_client,
	wayland_egl,
	wayland_protos,
	egl,
	glesv2,
	drm,
	gbm,
	libinput,
	xkbcommon,
	udev,
	pixman,
	math,
	rt,
]

libinput_ver = libinput.version().split('.')
add_project_arguments([
	'-DLIBINPUT_MAJOR=' + libinput_ver[0],
	'-DLIBINPUT_MINOR=' + libinput_ver[1],
	'-DLIBINPUT_PATCH=' + libinput_ver[2],
	'-I/usr/include/android',
], language: 'c')

subdir('protocol')
subdir('render')

subdir('backend')
subdir('types')
subdir('util')
subdir('xcursor')
subdir('xwayland')

subdir('include')

wlr_inc = include_directories('.', 'include')
proto_inc = include_directories('protocol')

symbols_file = 'wlroots.syms'
symbols_flag = '-Wl,--version-script,@0@/@1@'.format(meson.current_source_dir(), symbols_file)
lib_wlr = library(
	meson.project_name(), wlr_files,
	soversion: soversion,
	dependencies: wlr_deps,
	include_directories: [wlr_inc, proto_inc],
	install: true,
	link_args : symbols_flag,
	link_depends: symbols_file,
)

wlroots = declare_dependency(
	link_with: lib_wlr,
	dependencies: wlr_deps,
	include_directories: wlr_inc,
)

meson.override_dependency('wlroots', wlroots)

summary({
	'systemd': conf_data.get('WLR_HAS_SYSTEMD', 0),
	'elogind': conf_data.get('WLR_HAS_ELOGIND', 0),
	'libseat': conf_data.get('WLR_HAS_LIBSEAT', 0),
	'xwayland': conf_data.get('WLR_HAS_XWAYLAND', 0),
	'x11_backend': conf_data.get('WLR_HAS_X11_BACKEND', 0),
	'xcb-icccm': conf_data.get('WLR_HAS_XCB_ICCCM', 0),
	'xcb-errors': conf_data.get('WLR_HAS_XCB_ERRORS', 0),
})

if get_option('examples')
	subdir('examples')
endif

pkgconfig = import('pkgconfig')
pkgconfig.generate(lib_wlr,
	version: meson.project_version(),
	filebase: meson.project_name(),
	name: meson.project_name(),
	description: 'Wayland compositor library',
)
