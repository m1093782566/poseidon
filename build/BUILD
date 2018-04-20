package(default_visibility = ["//visibility:public"])

load("@io_bazel_rules_docker//docker:docker.bzl", "docker_image", "docker_bundle", "docker_push")
load("@io_kubernetes_build//defs:build.bzl", "release_filegroup")

filegroup(
    name = "package-srcs",
    srcs = glob(["**"]),
    tags = ["automanaged"],
)

filegroup(
    name = "all-srcs",
    srcs = [
        ":package-srcs",
        "//build/release-tars:all-srcs",
        "//build/visible_to:all-srcs",
    ],
    tags = ["automanaged"],
)

# Change the base image to hyperkube to reduce size, if need be.
docker_image(
    name = "poseidon-image",
    base = "@debian-hyperkube-base-amd64//image",
    cmd = ["/poseidon"],
    files = ["//cmd/poseidon"],
)

# This is to pack the image in the release tar also.
docker_bundle(
    name = "poseidon-amd64",
    images = {
        "gcr.io/google_containers/poseidon-amd64:{STABLE_DOCKER_TAG}": "poseidon-image",
    },
    stamp = True,
)

docker_push(
    name = "push-poseidon-image",
    image = ":poseidon-image",
    registry = "gcr.io",
    repository = "$(repository)",
    tag = "dev",
)

release_filegroup(
    name = "docker-artifacts",
    srcs = [":poseidon-amd64.tar"],
)

genrule(
    name = "os_package_version",
    outs = ["version"],
    cmd = """
grep ^STABLE_BUILD_SCM_REVISION bazel-out/stable-status.txt \
    | awk '{print $$2}' \
    | sed -e 's/^v//' -Ee 's/-([a-z]+)/~\\1/' -e 's/-/+/g' \
    >$@
""",
    stamp = 1,
)

# KUBE_SERVER_TARGETS
# No need to duplicate CLIENT_TARGETS or NODE_TARGETS here,
# since we include them in the actual build rule.
release_filegroup(
    name = "server-targets",
    srcs = [
        "//cmd/poseidon",
    ],
)
