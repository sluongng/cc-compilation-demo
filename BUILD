load("@rules_cc//cc:defs.bzl", "cc_binary")

[
    genrule(
        name = "create_source_{}".format(i),
        outs = ["source_{}.c".format(i)],
        cmd = """cat <<EOF > $@
#include <stdio.h>

int main()
{{
    printf("HEllo ");
    printf("Force {}");
    return 0;
}}
EOF
""".format(i),
    )
    for i in range(1, 200)
]

[
    cc_binary(
        name = "hello_world_{}".format(i),
        srcs = ["source_{}.c".format(i)],
    )
    for i in range(1, 200)
]
