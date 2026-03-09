# shaderc build.zig

A `build.zig` for [shaderc](https://github.com/google/shaderc), making it easy to use from Zig projects and to cross-compile. Provides both the `shaderc` library and the `glslc` command-line shader compiler as build artifacts.

## Usage

### 1. Add the dependency

```sh
zig fetch --save=shaderc git+https://github.com/diogok/shaderc
```

### 2. Use in your `build.zig`

Get the `glslc` artifact from the dependency and use it to compile GLSL shaders to SPIR-V during your build:

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const shaderc_dep = b.dependency("shaderc", .{});
    const glslc = shaderc_dep.artifact("glslc");

    // Compile a shader
    const spv = compileShader(b, glslc, b.path("shaders/my_shader.comp"), "my_shader");

    // Use the compiled SPIR-V as an embedded module
    const exe = b.addExecutable(.{ ... });
    exe.root_module.addAnonymousImport("my_shader_spv", .{ .root_source_file = spv });
}

fn compileShader(
    b: *std.Build,
    glslc: *std.Build.Step.Compile,
    source: std.Build.LazyPath,
    name: []const u8,
) std.Build.LazyPath {
    const cmd = b.addRunArtifact(glslc);
    cmd.addArgs(&.{ "--target-env=vulkan1.1", "-o" });
    const spv = cmd.addOutputFileArg(b.fmt("{s}.spv", .{name}));
    cmd.addFileArg(source);
    return spv;
}
```

You can also pass preprocessor defines to `glslc`:

```zig
const cmd = b.addRunArtifact(glslc);
cmd.addArgs(&.{ "--target-env=vulkan1.1", "-o" });
const spv = cmd.addOutputFileArg("my_shader.spv");
cmd.addArgs(&.{ "-DBATCH_SIZE=32", "-DUSE_FP16" });
cmd.addFileArg(source);
```

## License

MIT License

Copyright (c) 2025 Diogo Souza da Silva

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
