# SPDX-FileCopyrightText: 2022 Aetf <aetf@unlimited-code.works>
#
# SPDX-License-Identifier: MIT

manpages = [
  'kmscon.1.xml',
]

foreach man : manpages
  name = fs.replace_suffix(man, '')
  custom_target(
    name,
    command: [
      xsltproc,
      '--stringparam', 'man.authors.section.enabled', '0',
      '--stringparam', 'man.copyright.section.enabled', '0',
      '--stringparam', 'funcsynopsis.style', 'ansi',
      '--stringparam', 'man.output.quietly', '1',
      '--nonet',
      '-o', '@OUTPUT@',
      manpages_stylesheet,
      '@INPUT@'
    ],
    input: [
      f'man/@man@',
    ],
    output: name,
    build_by_default: true,
    install: true,
    install_dir: mandir / 'man1',
  )
endforeach

