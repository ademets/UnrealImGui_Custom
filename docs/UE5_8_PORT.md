# Custom ImGui UE 5.8 port

This feature branch starts at custom `c1cb1a0`, preserving the vendored Dear
ImGui 1.81 and both `ImGui` and `ImGUIBlueprintLibrary` modules. It is scoped
to the isolated dm-unreal UE5 work in
[dm-unreal #15](https://github.com/ademets/dm-unreal/issues/15). The original
4.27 consumer remains on `c1cb1a0`; this port has not been merged.

UE5 removed the `FStringClassReference` alias used by the reflected ImGui
settings. The port uses `FSoftClassPath`, retaining `ImGuiInputHandlerClass`
property name, reflected APIs, class/module names and native delegate
name. An actual a103 UE5.8.0 consumer build failed at UHT on that old type,
with natural exit 6 in 73.84 seconds. The independent judge checked the
actual 5.8.3 headers before the fix was shaped.

The next a103 UE5.8.0 consumer build (build02) failed naturally with exit 6
in 43.5041509 seconds. UHT reported at `ImGuiModuleSettings.h(171)`:
`MetaData MetaClass on ImGuiInputHandlerClass has value 'ImGuiInputHandler' which is not a valid long path name.`
It suggested `/Script/ImGui.ImGuiInputHandler`. The port replaces that
short metadata value with the fully qualified path: `ImGui.uplugin`
declares the `ImGui` module, and `ImGuiInputHandler.h` declares
`UImGuiInputHandler`. The property's name/type and reflected APIs stay
unchanged.

Slate positions are copied explicitly into FVector2f values, and TArray
shrink arguments use EAllowShrinking with the original Yes/No intent.
No rendering/input result follows from these compiler compatibility edits.

One non-reflected public C++ helper intentionally changes signature:
`Converter::FVector2DToFloat2(FVector2D, float* Result[2])` becomes
`Converter::FVector2DToFloat2(FVector2D, float Result[2])`. The old form
accepted `float**` and returned addresses of by-value vector fields;
those addresses were dangling, and UE5's double fields also make the
assignment illegal. The new form copies X/Y into caller-owned storage
with at least two floats. No in-repo source callers exist. External native
callers must migrate from an array of pointers to two-float storage; this
is a deliberate C++ signature/ABI change. It is not a reflected Blueprint
API change. No pointer cast or local-address return is used.

The post-metadata-fix consumer build and copied map proof are pending at this
checkpoint. The broader Blueprint helper lifetime/child-scope backlog in
[dm-unreal #12](https://github.com/ademets/dm-unreal/issues/12) remains open;
this narrow compile repair does not claim to resolve it.

Execution is explicitly pinned to GPT-6.1-sol max, fallback none. The parent
dm-hub session owns independent source and live-build judgment. See the
dm-unreal UE5_RUNBOOK for host, branch, source and receipt custody.
