# SPDX-FileCopyrightText: 2022 Aetf <aetf@unlimited-code.works>
#
# SPDX-License-Identifier: MIT

foreach name, deps : {
  'output': [uterm_deps],
  'vt': [uterm_deps],
  'input': [uterm_deps],
  'key': [],
}
  deps += [
    shl_deps,
    conf_deps,
    xkbcommon_deps,
  ]
  test_name = f'test_@name@'
  exe = executable(test_name, f'@test_name@.c',
    dependencies: deps,
  )
  test(test_name, exe)
endforeach
