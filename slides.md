---
# You can also start simply with 'default'
theme: default
title: SwiftUI Dark Mode
info: |
  ## How to implement Dark Mode in SwiftUI
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
overviewSnapshots: true
---

# Dark Mode in SwiftUI

How to implement dark mode for IOS Apps in SwiftUI

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/ngquyduc/kovalee-darkmode" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
layout: center
---

# Key aspects of SwiftUI's dark mode implementation

---
transition: slide-up
---

# 1. System Environment

SwiftUI uses an environment value called colorScheme to determine the current appearance:

```swift {|2,5}
struct ContentView: View {
    @Environment(\.colorScheme) var colorScheme
    
    var body: some View {
        Text("Current mode: \(colorScheme == .dark ? "Dark" : "Light")")
    }
}
```


---
transition: fade
---

# 2. Built-in Color Adaptation

Colors in SwiftUI automatically adapt to dark mode in several ways:

**System Colors**

```swift {|6,8,12,14}
struct SystemColorsView: View {
    var body: some View {
        VStack {
            // These colors automatically adapt
            Text("Primary")
                .foregroundStyle(.primary)  // Black in light, White in dark
            Text("Secondary")
                .foregroundStyle(.secondary)  // Dark gray in light, Light gray in dark
            
            // System Background colors
            Rectangle()
                .fill(.background)  // White in light, Black in dark
            Rectangle()
                .fill(.secondaryBackground)  // Light gray in light, Dark gray in dark
        }
    }
}
```

---
transition: fade
---

# 2. Built-in Color Adaptation

Colors in SwiftUI automatically adapt to dark mode in several ways:

**Asset Catalog Colors**

``` swift
// In Assets.xcassets, create a Color Set with:
// - Appearance: Any, Dark
// - Input Method: Color
// Then use it in code:
struct AssetColorView: View {
    var body: some View {
        Color("AccentColor") // Will automatically switch based on mode
    }
}
```

---
transition: slide-up
---

# 2. Built-in Color Adaptation

Colors in SwiftUI automatically adapt to dark mode in several ways:

**Semantic Colors**

```swift{|5,10,13}
struct SemanticColorsView: View {
    var body: some View {
        VStack {
            Label("Default Text", systemImage: "text.bubble")
                .foregroundStyle(.primary)
            
            Button("Action") {
                // action
            }
            .tint(.accentColor)  // System blue that adapts to dark mode
            
            Label("Warning", systemImage: "exclamationmark.triangle")
                .foregroundStyle(.red)  // System red that adapts
        }
    }
}
```

---
transition: slide-up
---

# 3. Manual Handling

When you need custom colors for each mode:

```swift{|5,6,15,16,17}
struct CustomColorView: View {
    @Environment(\.colorScheme) var colorScheme
    
    var backgroundColor: Color {
        colorScheme == .dark ? 
            Color(red: 0.2, green: 0.2, blue: 0.2) : Color(red: 0.9, green: 0.9, blue: 0.9)
    }
    
    var body: some View {
        VStack {
            Text("Hello")
                .background(backgroundColor)
            
            Text("World")
                .foregroundStyle(
                    colorScheme == .dark ? .white : .black
                )
            
            Text("Fixed Dark Mode")
                .preferredColorScheme(.dark)
        }
    }
}
```

---
transition: slide-up
---

# 4. Overriding System Mode
For Entire View Hierarchy

```swift{|8}
struct OverrideView: View {
    var body: some View {
        NavigationView {
            List {
                Text("Always Dark Mode")
            }
        }
        .preferredColorScheme(.dark) // Forces dark mode
    }
}
```

---
transition: fade
---

# 5. App-Level Config

In your app's main entry point:

```swift{|3,12,13,14,15,16,17,18,19,20,21}
@main
struct MyApp: App {
    @AppStorage("userInterfaceStyle") var userInterfaceStyle: String = "system"
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .preferredColorScheme(colorScheme)
        }
    }
    
    var colorScheme: ColorScheme? {
        switch userInterfaceStyle {
        case "light":
            return .light
        case "dark":
            return .dark
        default:
            return nil  // Follow system
        }
    }
}
```

---

# 5. App-Level Config

In your app's main entry point:

```swift
// Settings view to change the mode
struct SettingsView: View {
    @AppStorage("userInterfaceStyle") var userInterfaceStyle: String = "system"
    
    var body: some View {
        Picker("Appearance", selection: $userInterfaceStyle) {
            Text("System").tag("system")
            Text("Light").tag("light")
            Text("Dark").tag("dark")
        }
    }
}
```

---

# SwiftUI Default Approach Limitations

1. Scattered Color Definitions
2. No Central Color Management
3. Limited Theme Variations
4. Complex Color Scheme Changes

--- 
layout: image

image: ./assets/components-ios.png
---

---
layout: center
transition: slide-up
---

# New ThemeProtocol


---
transition: slide-up
---

# Step 1: Color Sets

First, we introduce a ColorSet struct to group colors:

```swift
public struct ColorSet {
    let primary: Color
    let secondary: Color
    let tertiary: Color
    let background: Color
    let secondaryBackground: Color
    let text: Color
    let error: Color
    let pro: Color
    
    public init(
      ...
    ) {
      ...
    }
}
```

---
transition: slide-up
---

# Step 2: Theme Mode

Adding theme mode support:

```swift
public enum ThemeMode {
    case light
    case dark
    case system
}

public protocol DynamicColorProtocol {
    var lightColors: ColorSet { get }
    var darkColors: ColorSet { get }
    var currentMode: ThemeMode { get }
    
    func getColor(_ keyPath: KeyPath<ColorSet, Color>) -> Color
}
```

---
transition: fade
---

# Step 3: Dynamic Colors Implementation

Implementing the dynamic color system:

```swift
public class DynamicColors: DynamicColorProtocol, ObservableObject {
    public let lightColors: ColorSet
    public let darkColors: ColorSet
    @Published public var currentMode: ThemeMode
    
    @Environment(\.colorScheme) private var systemColorScheme
    
    public init(lightColors: ColorSet, darkColors: ColorSet, initialMode: ThemeMode = .system) {
        self.lightColors = lightColors
        self.darkColors = darkColors
        self.currentMode = initialMode
    }
```

---
transition: slide-up
---

# Step 3: Dynamic Colors Implementation

Implementing the dynamic color system:

```swift
    public func getColor(_ keyPath: KeyPath<ColorSet, Color>) -> Color {
        switch currentMode {
        case .light:
            return lightColors[keyPath: keyPath]
        case .dark:
            return darkColors[keyPath: keyPath]
        case .system:
            return systemColorScheme == .dark ? 
                darkColors[keyPath: keyPath] : 
                lightColors[keyPath: keyPath]
        }
    }
}
```

---
transition: fade
---

# Step 4: Updated Color Protocol

Modifying the original ColorProtocol to support theme switching:

````md magic-move
```swift
public protocol ColorProtocol {
    var primary: Color { get }
    var secondary: Color { get }
    var tertiary: Color { get }
    var background: Color { get }
    var secondaryBackground: Color { get }
    var text: Color { get }
    var error: Color { get }
    var pro: Color { get }
}
```
```swift
public protocol ColorProtocol {
    var primary: Color { get }
    var secondary: Color { get }
    var tertiary: Color { get }
    var background: Color { get }
    var secondaryBackground: Color { get }
    var text: Color { get }
    var error: Color { get }
    var pro: Color { get }
    
    // New method for theme switching
    func switchToMode(_ mode: ThemeMode)
}
```
````

---
transition: slide-up
---

# Step 4: Updated Color Protocol

Modifying the original ColorProtocol to support theme switching:

```swift
public class DynamicThemeColors: ColorProtocol, ObservableObject {
    private let dynamicColors: DynamicColors
    
    public init(dynamicColors: DynamicColors) {
        self.dynamicColors = dynamicColors
    }
    
    public var primary: Color { dynamicColors.getColor(\.primary) }
    public var secondary: Color { dynamicColors.getColor(\.secondary) }
    public var tertiary: Color { dynamicColors.getColor(\.tertiary) }
    public var background: Color { dynamicColors.getColor(\.background) }
    public var secondaryBackground: Color { dynamicColors.getColor(\.secondaryBackground) }
    public var text: Color { dynamicColors.getColor(\.text) }
    public var error: Color { dynamicColors.getColor(\.error) }
    public var pro: Color { dynamicColors.getColor(\.pro) }
    
    public func switchToMode(_ mode: ThemeMode) {
        dynamicColors.currentMode = mode
    }
}
```

---

# Final Implementation: CustomTheme

Putting it all together in the CustomTheme class:

````md magic-move
```swift
public protocol ThemeProtocol {
    /// Color theme components adhering to `ColorProtocol`.
    var colors: ColorProtocol { get }
    /// Typography settings adhering to `TypographyProtocol`.
    var typography: TypographyProtocol { get }
    /// Spacing metrics adhering to `SpacingProtocol`.
    var spacing: SpacingProtocol { get }
}
```
```swift
public class CustomTheme: ThemeProtocol, ObservableObject {
    @Published public var colors: ColorProtocol
    public var typography: TypographyProtocol
    public var spacing: SpacingProtocol
    
    public init(
        colors: ColorProtocol,
        typography: TypographyProtocol = BasicTypography(),
        spacing: SpacingProtocol = BasicSpacing()
    ) {
        self.colors = colors
        self.typography = typography
        self.spacing = spacing
    }
}
```
```swift
// Environment setup
private struct ThemeKey: EnvironmentKey {
    static let defaultValue: CustomTheme = CustomTheme(
        colors: DynamicThemeColors(
            dynamicColors: DynamicColors(
                lightColors: ColorSet(/* light mode colors */),
                darkColors: ColorSet(/* dark mode colors */)
            )
        )
    )
}
```
````

---
transition: fade
---

# Usage Example

How to use the new theme system in your app:

```swift
struct MyApp: App {
    @StateObject private var theme: CustomTheme = {
        // Define your color sets
        let lightColors = ColorSet(
            ...
        )
        
        let darkColors = ColorSet(
            ...
        )
        
        let dynamicColors = DynamicColors(
            lightColors: lightColors,
            darkColors: darkColors,
            initialMode: .system
        )
        
        return CustomTheme(
            colors: DynamicThemeColors(dynamicColors: dynamicColors)
        )
    }()
```
---
transition: fade
---

# Usage Example

How to use the new theme system in your app:

```swift    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.theme, theme)
        }
    }
}
```

---
layout: center
---
# DEMO WITH JOURNALING APP

---
layout: center
---

# [PULL REQUEST](https://github.com/cotyapps/Components-iOS/pull/96)

---
layout: center
---

# THANK YOU!
