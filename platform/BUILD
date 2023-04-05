platform(
    name = "platform_linux",
    constraint_values = [
        "@platforms//cpu:x86_64",
        "@platforms//os:linux",
        "@bazel_tools//tools/cpp:clang",
    ],
    exec_properties = {
        "OSFamily": "Linux",
        "container-image": "docker://gcr.io/flame-public/rbe-ubuntu20-04:latest",
        "recycle-runner": "true",
    },
)
