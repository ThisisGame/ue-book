when compile ue4,there is a SharedPCH.UnrealEd.NoShadow.cpp,what is it ?

In Unreal Engine 4 (UE4), PCH stands for Precompiled Header. Precompiled headers are a performance optimization that compile a large chunk of C++ code into a form that's faster to process for the compiler. This can significantly speed up compile times, especially in large codebases like UE4.

The SharedPCH.UnrealEd.NoShadow.cpp file is one of these precompiled headers. The UnrealEd part of the name suggests that this precompiled header is used by the Unreal Editor part of the engine.

The NoShadow part of the name is a bit more specific to UE4's build system. UE4 has a concept of "shadow" builds, which are additional builds of the engine that are done with different settings. A "NoShadow" PCH is one that's used when building the engine without these shadow builds.


ThisisGame: my compute is old,while build SharedPCH.UnrealEd.NoShadow.cpp,my compute reboot,what can i do?

GitHub Copilot: Building Unreal Engine 4 (UE4) can be resource-intensive and may cause issues on older hardware. Here are a few suggestions:

1. **Increase Virtual Memory**: Sometimes, insufficient virtual memory can cause the system to reboot. Increase the size of the paging file (virtual memory) on your system.

2. **Close Unnecessary Applications**: Close all unnecessary applications running in the background to free up system resources.

3. **Update Your System**: Make sure your operating system and drivers are up-to-date. This can sometimes help with stability issues.

4. **Build in Smaller Chunks**: Instead of building the entire engine at once, try building it in smaller parts. This can be less taxing on your system.

5. **Upgrade Your Hardware**: If possible, consider upgrading your hardware. More RAM and a faster CPU can significantly improve build times and system stability.

6. **Use a Lighter IDE**: If you're using a heavy IDE like Visual Studio, you might want to switch to a lighter one like Visual Studio Code or even a text editor.

Remember to always save your work frequently to prevent data loss in case of system reboots.

ThisisGame: can i close pch in ue4?

GitHub Copilot: Yes, you can disable the use of Precompiled Headers (PCH) in Unreal Engine 4 (UE4). However, it's important to note that this might increase your compilation times, as PCHs are used to speed up the build process.

To disable PCHs, you need to modify the build rules for your project. Here's how you can do it:

1. Open the `.Build.cs` file for your project. This file is usually located in the `Source` directory of your project.

2. Find the `PCHUsage` property in the `TargetRules` class and set it to `PCHUsageMode.NoPCHs`.

Here's an example:

```cpp
public class YourProjectTarget : TargetRules
{
    public YourProjectTarget(TargetInfo Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.NoPCHs;
        // other settings...
    }
}
```

Replace `YourProjectTarget` with the actual name of your project.

Remember to recompile your project after making these changes.

ThisisGame: how close all module pch?

GitHub Copilot: To disable precompiled headers (PCH) for all modules in Unreal Engine 4 (UE4), you can modify the build configuration in the `.Target.cs` file of your project. Here's how you can do it:

1. Open the `.Target.cs` file for your project.
2. Set `bUsePCHFiles` to `false`.

Here's an example of what the code might look like:

```csharp
public class YourProjectTarget : TargetRules
{
    public YourProjectTarget(TargetInfo Target) : base(Target)
    {
        Type = TargetType.Game;
        DefaultBuildSettings = BuildSettingsVersion.V2;
        bUsePCHFiles = false;

        // Other configuration settings...
    }
}
```

Please note that disabling PCHs can significantly increase your compile times, as the compiler will need to parse all included headers for each source file, instead of just loading the precompiled headers. It's generally recommended to use PCHs unless you have a specific reason not to.

Next steps:
- Verify the impact on compile times after disabling PCHs.
- Consider other optimization techniques if compile times are too long.