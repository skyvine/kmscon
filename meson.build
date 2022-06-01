# SPDX-FileCopyrightText: 2022 Aetf <aetf@unlimited-code.works>
#
# SPDX-License-Identifier: MIT

project('kmscon', 'c',
  version: '9.0.0',
  license: 'MIT',
  # meson 0.58: f-string
  # meson 0.62: dependency libdl
  meson_version: '>=0.62.0',
  default_options: [
    'warning_level=1',
    'werror=true',
    'buildtype=debugoptimized',
    'c_std=gnu99',
  ]
)

add_project_arguments(
  '-ffast-math',
  '-fno-strict-aliasing',
  '-ffunction-sections',
  '-fdata-sections',
  '-fstack-protector',
  '-fvisibility=hidden',
  '-D_GNU_SOURCE',
  '-D_POSIX_C_SOURCE=200809L',
  language: 'c'
)

#
# Directory structure
#
prefix = get_option('prefix')
bindir = get_option('bindir')
libexecdir = get_option('libexecdir') / meson.project_name()
mandir = get_option('mandir')
moduledir = get_option('libdir') / meson.project_name()

#
# Required dependencies
#
python = import('python').find_installation()
fs = import('fs')

xkbcommon_deps = dependency('xkbcommon', version: '>=0.5.0')
libtsm_deps = dependency('libtsm', version: '>=4.0.0')
libudev_deps = dependency('libudev', version: '>=172')
libdrm_deps = dependency('libdrm')
dl_deps = dependency('dl')
threads_deps = dependency('threads')

#
# Dependencies
# This unconditionally checks for all dependencies even if they are disabled. We
# later look whether all required depedencies are met and finish the
# configuration.
#
libsystemd_deps = dependency('libsystemd', required: false)
gbm_deps = dependency('gbm', required: false)
egl_deps = dependency('egl', required: false)
glesv2_deps = dependency('glesv2', required: false)
pango_deps = dependency('pangoft2', required: false)
pixman_deps = dependency('pixman-1', required: false)
xsltproc = find_program('xsltproc', native: true, required: false)

#
# Handle feature options
# This combines feature option inputs with dependency requirements to derive the
# final set of values that will determine what gets built.
# This will also store the information in a configuration header.
#
sections = {
  'video': 'Video Backends',
  'font': 'Font Backends',
  'renderer': 'Renderers',
  'session': 'Session Types',
}
config = configuration_data()
foreach name, reqs : {
  'multi_seat': [libsystemd_deps],
  'video_fbdev': [],
  'video_drm2d': [],
  'video_drm3d': [gbm_deps, egl_deps, glesv2_deps],
  'renderer_bbulk': [],
  'renderer_gltex': [glesv2_deps],
  'renderer_pixman': [pixman_deps],
  'font_unifont': [],
  'font_pango': [pango_deps],
  'session_dummy': [],
  'session_terminal': [],
}
  found = true
  foreach req : reqs
    found = found and req.found()
  endforeach
  enabled = get_option(name).require(found).allowed()
  upper_name = name.to_upper()
  # set a variable for later use
  set_variable(f'enable_@name@', enabled)
  # set in config header for code use
  config.set(f'BUILD_ENABLE_@upper_name@', enabled)
  # summary output
  summary(name, enabled, section: sections.get(name.split('_')[0], 'Miscellaneous'))
endforeach

config.set_quoted('BUILD_MODULE_DIR', prefix / moduledir)

# Make all files include "config.h" by default. This shouldn't cause any
# problems and we cannot forget to include it anymore.
config_h = configure_file(configuration: config, output: 'config.h')
abs_config_h = meson.current_build_dir() / '@0@'.format(config_h)
add_project_arguments('-include', abs_config_h, language: 'c')

#
# Miscellaneous checks
# Not directly related to dependencies but
#
manpages_stylesheet = 'http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl'
have_manpages_stylesheet = false
if xsltproc.found()
  have_manpages_stylesheet = run_command(xsltproc, '--nonet', manpages_stylesheet, check: false).returncode() == 0
endif
enable_docs = get_option('docs').require(xsltproc.found() and have_manpages_stylesheet).allowed()

# Additionally check if glesv2 is needed
enable_glesv2 = enable_video_drm3d and enable_renderer_gltex

#
# Other configuration output
# Show configuration to the user so they can check whether everything was
# configured as expected.
#
summary({
  'prefix': prefix,
  'libexecdir': libexecdir,
  'bindir': bindir,
  'moduledir': moduledir,
  'mandir': mandir,
}, section: 'Directories')
summary({
  'extra-debug': get_option('extra_debug'),
  'tests': get_option('tests'),
  'docs': enable_docs,
}, section: 'Miscellaneous')

#
# Putting code together
#
subdir('external')
subdir('src')

if get_option('tests')
  subdir('tests')
endif
if enable_docs
  subdir('docs')
endif

#
# Process other files
#
dirs_info = configuration_data()
dirs_info.set('libexecdir', prefix / libexecdir)
dirs_info.set('bindir', prefix / bindir)
foreach filename, kwargs : {
  'scripts/kmscon.in': {
    'install_dir': bindir,
    'install_mode': 'rwxr-xr-x',
  },
  'docs/kmscon.service.in': {},
  'docs/kmsconvt@.service.in': {},
}
  install_data(configure_file(input: filename, output: '@BASENAME@', configuration: dirs_info),
    kwargs: kwargs,
  )
endforeach
