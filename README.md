# ztracy v0.13.0 - performance markers for Tracy 0.11.1

Initial Zig bindings created by [Martin Wickham](https://github.com/SpexGuy/Zig-Tracy)

Entirely copied from [zig-gamedev](https://github.com/zig-gamedev/zig-gamedev/). I take no credit for this repository.

## Getting started

Copy `ztracy` to a subdirectory of your project and add the following to your `build.zig.zon` .dependencies:

```zig
    .ztracy = .{ .path = "libs/ztracy" },
```

Then in your `build.zig` add:

```zig
pub fn build(b: *std.Build) void {
    const options = .{
        .enable_ztracy = b.option(
            bool,
            "enable_ztracy",
            "Enable Tracy profile markers",
        ) orelse false,
        .enable_fibers = b.option(
            bool,
            "enable_fibers",
            "Enable Tracy fiber support",
        ) orelse false,
        .on_demand = b.option(
            bool,
            "on_demand",
            "Build tracy with TRACY_ON_DEMAND",
        ) orelse false,
    };

    const exe = b.addExecutable(.{ ... });

    const ztracy = b.dependency("ztracy", .{
        .enable_ztracy = options.enable_ztracy,
        .enable_fibers = options.enable_fibers,
        .on_demand = options.on_demand,
    });
    exe.root_module.addImport("ztracy", ztracy.module("root"));
    exe.linkLibrary(ztracy.artifact("tracy"));
}
```

Now in your code you may import and use `ztracy`. To build your project with Tracy enabled run:

`zig build -Denable_ztracy=true`

```zig
const ztracy = @import("ztracy");

pub fn main() !void {
    {
        const tracy_zone = ztracy.ZoneNC(@src(), "Compute Magic", 0x00_ff_00_00);
        defer tracy_zone.End();
        ...
    }
}
```

## Async "Fibers" support

Tracy has support for marking fibers (also called green threads,
coroutines, and other forms of cooperative multitasking). This support requires
an additional option passed through when compiling the Tracy library, so:

```zig
    ...
    const optimize = b.standardOptimizeOption(.{});
    const target = b.standardTargetOptions(.{});

    const ztracy_pkg = ztracy.package(b, target, optimize, .{
        .options = .{ .enable_ztracy = true, .enable_fibers = true },
    });

    ztracy_pkg.link(exe);
```
