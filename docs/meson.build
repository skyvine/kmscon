# SPDX-FileCopyrightText: 2022 Aetf <aetf@unlimited-code.works>
#
# SPDX-License-Identifier: MIT

custom_target(
  'manpages',
  command: [
    xsltproc,
    '--stringparam', 'man.authors.section.enabled', '0',
    '--stringparam', 'man.copyright.section.enabled', '0',
    '--stringparam', 'funcsynopsis.style', 'ansi',
    '--stringparam', 'man.output.quietly', '1',
    '--nonet',
    '-o', '@OUTPUT@',
    '@INPUT@'
  ],
  input: [
    'man/kmscon.xml'
  ],
  output: 'man/',
  build_by_default: true,
  install: true,
  install_dir: mandir / 'man1',
)

