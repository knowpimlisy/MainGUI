--Theme
local http = game:GetService('HttpService') 

repeat wait()
until game:IsLoaded()
getgenv().AutoSecondWorld = false
getgenv().AutoThirdworld = false
getgenv().TextColorChange = Color3.fromRGB(0, 128, 255)
getgenv().AutoKen = false
getgenv().FastAttack = true
getgenv().AutoBuso = true
getgenv().Magnet = true
getgenv().MobAuraBring = true
getgenv().Distance = 15
getgenv().MagnetDistance = 300
getgenv().FastAttackCD = 0
getgenv().AutoFarmClick = true
getgenv().SaveSetting = {


}

getgenv().NoAutoJoinDiscord = false
local filename1 = "NoAutoJoinDiscord.txt";
function LoadSetting1()
   print("Loading Data")
   local HttpService = game:GetService("HttpService");
   if (readfile and isfile and isfile(filename1)) then
      getgenv().NoAutoJoinDiscord = HttpService:JSONDecode(readfile(filename1));
   end
end
LoadSetting1()
print(getgenv().NoAutoJoinDiscord)
local http = game:GetService('HttpService') 
local req =  http_request or request or HttpPost or syn.request -- get request

---
if getgenv().Magnet then
    Mode = "Magnet"
else
    Mode = "Back"
end
-- loading --
while true do wait()
	if game.Players.LocalPlayer then
		wait(5)
		break
	end
end
if game.Players.LocalPlayer then
   wait(3)
   local MyUi = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Pirates.Frame.ViewportFrame.TextButton -- 
   for i,v in pairs(getconnections(MyUi.MouseButton1Click)) do
   v.Function()
   end
   end
local VirtualUser=game:service'VirtualUser'
-- ui -- 
local ui = game:GetService("CoreGui"):FindFirstChild("Island| Blox Fruit (Released)")
if ui then
   ui:Destroy()
end
-- init
local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

-- services
local input = game:GetService("UserInputService")
local run = game:GetService("RunService")
local tween = game:GetService("TweenService")
local tweeninfo = TweenInfo.new

-- additional
local utility = {}

-- themes
local objects = {}
local themes = {
Background = Color3.fromRGB(15,15,15), 
Glow = Color3.fromRGB(0, 0, 0), 
Accent = Color3.fromRGB(10, 10, 10), 
LightContrast = Color3.fromRGB(20, 20, 20), 
DarkContrast = Color3.fromRGB(10,10,10),  
TextColor = Color3.fromRGB(0, 0, 255)
}

do
function utility:Create(instance, properties, children)
   local object = Instance.new(instance)

   for i, v in pairs(properties or {}) do
      object[i] = v

      if typeof(v) == "Color3" then -- save for theme changer later
         local theme = utility:Find(themes, v)

         if theme then
            objects[theme] = objects[theme] or {}
            objects[theme][i] = objects[theme][i] or setmetatable({}, {_mode = "k"})

            table.insert(objects[theme][i], object)
         end
      end
   end

   for i, module in pairs(children or {}) do
      module.Parent = object
   end

   return object
end

function utility:Tween(instance, properties, duration, ...)
   tween:Create(instance, tweeninfo(duration, ...), properties):Play()
end

function utility:Wait()
   run.RenderStepped:Wait()
   return true
end

function utility:Find(table, value) -- table.find doesn't work for dictionaries
   for i, v in  pairs(table) do
      if v == value then
         return i
      end
   end
end

function utility:Sort(pattern, values)
   local new = {}
   pattern = pattern:lower()

   if pattern == "" then
      return values
   end

   for i, value in pairs(values) do
      if tostring(value):lower():find(pattern) then
         table.insert(new, value)
      end
   end

   return new
end

function utility:Pop(object, shrink)
   local clone = object:Clone()

   clone.AnchorPoint = Vector2.new(0.5, 0.5)
   clone.Size = clone.Size - UDim2.new(0, shrink, 0, shrink)
   clone.Position = UDim2.new(0.5, 0, 0.5, 0)

   clone.Parent = object
   clone:ClearAllChildren()

   object.ImageTransparency = 1
   utility:Tween(clone, {Size = object.Size}, 0.2)

   spawn(function()
      wait(0.2)

      object.ImageTransparency = 0
      clone:Destroy()
   end)

   return clone
end

function utility:InitializeKeybind()
   self.keybinds = {}
   self.ended = {}

   input.InputBegan:Connect(function(key)
      if self.keybinds[key.KeyCode] then
         for i, bind in pairs(self.keybinds[key.KeyCode]) do
            bind()
         end
      end
   end)

   input.InputEnded:Connect(function(key)
      if key.UserInputType == Enum.UserInputType.MouseButton1 then
         for i, callback in pairs(self.ended) do
            callback()
         end
      end
   end)
end

function utility:BindToKey(key, callback)

   self.keybinds[key] = self.keybinds[key] or {}

   table.insert(self.keybinds[key], callback)

   return {
      UnBind = function()
         for i, bind in pairs(self.keybinds[key]) do
            if bind == callback then
               table.remove(self.keybinds[key], i)
            end
         end
      end
   }
end

function utility:KeyPressed() -- yield until next key is pressed
   local key = input.InputBegan:Wait()

   while key.UserInputType ~= Enum.UserInputType.Keyboard	 do
      key = input.InputBegan:Wait()
   end

   wait() -- overlapping connection

   return key
end

function utility:DraggingEnabled(frame, parent)

   parent = parent or frame

   -- stolen from wally or kiriot, kek
   local dragging = false
   local dragInput, mousePos, framePos

   frame.InputBegan:Connect(function(input)
      if input.UserInputType == Enum.UserInputType.MouseButton1 then
         dragging = true
         mousePos = input.Position
         framePos = parent.Position

         input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
               dragging = false
            end
         end)
      end
   end)

   frame.InputChanged:Connect(function(input)
      if input.UserInputType == Enum.UserInputType.MouseMovement then
         dragInput = input
      end
   end)

   input.InputChanged:Connect(function(input)
      if input == dragInput and dragging then
         local delta = input.Position - mousePos
         parent.Position  = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
      end
   end)

end

function utility:DraggingEnded(callback)
   table.insert(self.ended, callback)
end

end

-- classes

local library = {} -- main
local page = {}
local section = {}

do
library.__index = library
page.__index = page
section.__index = section

-- new classes

function library.new(title)
   local container = utility:Create("ScreenGui", {
      Name = title,
      Parent = game.CoreGui
   }, {
      utility:Create("ImageLabel", {
         Name = "Main",
         BackgroundTransparency = 1,
         Position = UDim2.new(0.25, 0, 0.052435593, 0),
         Size = UDim2.new(0, 511, 0, 428),
         Image = "rbxassetid://4641149554",
         ImageColor3 = themes.Background,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(4, 4, 296, 296)
      }, {
         utility:Create("ImageLabel", {
            Name = "Glow",
            BackgroundTransparency = 1,
            Position = UDim2.new(0, -15, 0, -15),
            Size = UDim2.new(1, 30, 1, 30),
            ZIndex = 0,
            Image = "rbxassetid://5028857084",
            ImageColor3 = themes.Glow,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(24, 24, 276, 276)
         }),
         utility:Create("ImageLabel", {
            Name = "Pages",
            BackgroundTransparency = 1,
            ClipsDescendants = true,
            Position = UDim2.new(0, 0, 0, 38),
            Size = UDim2.new(0, 126, 1, -38),
            ZIndex = 3,
            Image = "rbxassetid://5012534273",
            ImageColor3 = themes.DarkContrast,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(4, 4, 296, 296)
         }, {
            utility:Create("ScrollingFrame", {
               Name = "Pages_Container",
               Active = true,
               BackgroundTransparency = 1,
               Position = UDim2.new(0, 0, 0, 10),
               Size = UDim2.new(1, 0, 1, -20),
               CanvasSize = UDim2.new(0, 0, 0, 314),
               ScrollBarThickness = 0
            }, {
               utility:Create("UIListLayout", {
                  SortOrder = Enum.SortOrder.LayoutOrder,
                  Padding = UDim.new(0, 10)
               })
            })
         }),
         utility:Create("ImageLabel", {
            Name = "TopBar",
            BackgroundTransparency = 1,
            ClipsDescendants = true,
            Size = UDim2.new(1, 0, 0, 38),
            ZIndex = 5,
            Image = "rbxassetid://4595286933",
            ImageColor3 = themes.Accent,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(4, 4, 296, 296)
         }, {
            utility:Create("TextLabel", { -- title
               Name = "Title",
               AnchorPoint = Vector2.new(0, 0.5),
               BackgroundTransparency = 1,
               Position = UDim2.new(0, 12, 0, 19),
               Size = UDim2.new(1, -46, 0, 16),
               ZIndex = 5,
               Font = Enum.Font.GothamBold,
               Text = title,
               TextColor3 = themes.TextColor,
               TextSize = 14,
               TextXAlignment = Enum.TextXAlignment.Left
            })
         })
      })
   })

   utility:InitializeKeybind()
   utility:DraggingEnabled(container.Main.TopBar, container.Main)

   return setmetatable({
      container = container,
      pagesContainer = container.Main.Pages.Pages_Container,
      pages = {}
   }, library)
end

function page.new(library, title, icon)
   local button = utility:Create("TextButton", {
      Name = title,
      Parent = library.pagesContainer,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Size = UDim2.new(1, 0, 0, 26),
      ZIndex = 3,
      AutoButtonColor = false,
      Font = Enum.Font.Gotham,
      Text = "",
      TextSize = 14
   }, {
      utility:Create("TextLabel", {
         Name = "Title",
         AnchorPoint = Vector2.new(0, 0.5),
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 40, 0.5, 0),
         Size = UDim2.new(0, 76, 1, 0),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.65,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      icon and utility:Create("ImageLabel", {
         Name = "Icon", 
         AnchorPoint = Vector2.new(0, 0.5),
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 12, 0.5, 0),
         Size = UDim2.new(0, 16, 0, 16),
         ZIndex = 3,
         Image = "rbxassetid://" .. tostring(icon),
         ImageColor3 = themes.TextColor,
         ImageTransparency = 0.64
      }) or {}
   })

   local container = utility:Create("ScrollingFrame", {
      Name = title,
      Parent = library.container.Main,
      Active = true,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Position = UDim2.new(0, 134, 0, 46),
      Size = UDim2.new(1, -142, 1, -56),
      CanvasSize = UDim2.new(0, 0, 0, 466),
      ScrollBarThickness = 3,
      ScrollBarImageColor3 = themes.DarkContrast,
      Visible = false
   }, {
      utility:Create("UIListLayout", {
         SortOrder = Enum.SortOrder.LayoutOrder,
         Padding = UDim.new(0, 10)
      })
   })

   return setmetatable({
      library = library,
      container = container,
      button = button,
      sections = {}
   }, page)
end

function section.new(page, title)
   local container = utility:Create("ImageLabel", {
      Name = title,
      Parent = page.container,
      BackgroundTransparency = 1,
      Size = UDim2.new(1, -10, 0, 28),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.LightContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(4, 4, 296, 296),
      ClipsDescendants = true
   }, {
      utility:Create("Frame", {
         Name = "Container",
         Active = true,
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Position = UDim2.new(0, 8, 0, 8),
         Size = UDim2.new(1, -16, 1, -16)
      }, {
         utility:Create("TextLabel", {
            Name = "Title",
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 20),
            ZIndex = 2,
            Font = Enum.Font.GothamSemibold,
            Text = title,
            TextColor3 = themes.TextColor,
            TextSize = 12,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextTransparency = 1
         }),
         utility:Create("UIListLayout", {
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 4)
         })
      })
   })

   return setmetatable({
      page = page,
      container = container.Container,
      colorpickers = {},
      modules = {},
      binds = {},
      lists = {},
   }, section) 
end

function library:addPage(...)

   local page = page.new(self, ...)
   local button = page.button

   table.insert(self.pages, page)

   button.MouseButton1Click:Connect(function()
      self:SelectPage(page, true)
   end)

   return page
end

function page:addSection(...)
   local section = section.new(self, ...)

   table.insert(self.sections, section)

   return section
end

-- functions

function library:setTheme(theme, color3)
   themes[theme] = color3

   for property, objects in pairs(objects[theme]) do
      for i, object in pairs(objects) do
         if not object.Parent or (object.Name == "Button" and object.Parent.Name == "ColorPicker") then
            objects[i] = nil -- i can do this because weak tables :D
         else
            object[property] = color3
         end
      end
   end
end

function library:toggle()

   if self.toggling then
      return
   end

   self.toggling = true

   local container = self.container.Main
   local topbar = container.TopBar

   if self.position then
      utility:Tween(container, {
         Size = UDim2.new(0, 511, 0, 428),
         Position = self.position
      }, 0.2)
      wait(0.2)

      utility:Tween(topbar, {Size = UDim2.new(1, 0, 0, 38)}, 0.2)
      wait(0.2)

      container.ClipsDescendants = false
      self.position = nil
   else
      self.position = container.Position
      container.ClipsDescendants = true

      utility:Tween(topbar, {Size = UDim2.new(1, 0, 1, 0)}, 0.2)
      wait(0.2)

      utility:Tween(container, {
         Size = UDim2.new(0, 511, 0, 0),
         Position = self.position + UDim2.new(0, 0, 0, 428)
      }, 0.2)
      wait(0.2)
   end

   self.toggling = false
end

-- new modules

function library:Notify(title, text, callback)

   -- overwrite last notification
   if self.activeNotification then
      self.activeNotification = self.activeNotification()
   end

   -- standard create
   local notification = utility:Create("ImageLabel", {
      Name = "Notification",
      Parent = self.container,
      BackgroundTransparency = 1,
      Size = UDim2.new(0, 200, 0, 60),
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.Background,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(4, 4, 296, 296),
      ZIndex = 3,
      ClipsDescendants = true
   }, {
      utility:Create("ImageLabel", {
         Name = "Flash",
         Size = UDim2.new(1, 0, 1, 0),
         BackgroundTransparency = 1,
         Image = "rbxassetid://4641149554",
         ImageColor3 = themes.TextColor,
         ZIndex = 5
      }),
      utility:Create("ImageLabel", {
         Name = "Glow",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, -15, 0, -15),
         Size = UDim2.new(1, 30, 1, 30),
         ZIndex = 2,
         Image = "rbxassetid://5028857084",
         ImageColor3 = themes.Glow,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(24, 24, 276, 276)
      }),
      utility:Create("TextLabel", {
         Name = "Title",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0, 8),
         Size = UDim2.new(1, -40, 0, 16),
         ZIndex = 4,
         Font = Enum.Font.GothamSemibold,
         TextColor3 = themes.TextColor,
         TextSize = 14.000,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("TextLabel", {
         Name = "Text",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 1, -24),
         Size = UDim2.new(1, -40, 0, 16),
         ZIndex = 4,
         Font = Enum.Font.Gotham,
         TextColor3 = themes.TextColor,
         TextSize = 12.000,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("ImageButton", {
         Name = "Accept",
         BackgroundTransparency = 1,
         Position = UDim2.new(1, -26, 0, 8),
         Size = UDim2.new(0, 16, 0, 16),
         Image = "rbxassetid://5012538259",
         ImageColor3 = themes.TextColor,
         ZIndex = 4
      }),
      utility:Create("ImageButton", {
         Name = "Decline",
         BackgroundTransparency = 1,
         Position = UDim2.new(1, -26, 1, -24),
         Size = UDim2.new(0, 16, 0, 16),
         Image = "rbxassetid://5012538583",
         ImageColor3 = themes.TextColor,
         ZIndex = 4
      })
   })

   -- dragging
   utility:DraggingEnabled(notification)

   -- position and size
   title = title or "Notification"
   text = text or ""

   notification.Title.Text = title
   notification.Text.Text = text

   local padding = 10
   local textSize = game:GetService("TextService"):GetTextSize(text, 12, Enum.Font.Gotham, Vector2.new(math.huge, 16))

   notification.Position = library.lastNotification or UDim2.new(0, padding, 1, -(notification.AbsoluteSize.Y + padding))
   notification.Size = UDim2.new(0, 0, 0, 60)

   utility:Tween(notification, {Size = UDim2.new(0, textSize.X + 70, 0, 60)}, 0.2)
   wait(0.2)

   notification.ClipsDescendants = false
   utility:Tween(notification.Flash, {
      Size = UDim2.new(0, 0, 0, 60),
      Position = UDim2.new(1, 0, 0, 0)
   }, 0.2)

   -- callbacks
   local active = true
   local close = function()

      if not active then
         return
      end

      active = false
      notification.ClipsDescendants = true

      library.lastNotification = notification.Position
      notification.Flash.Position = UDim2.new(0, 0, 0, 0)
      utility:Tween(notification.Flash, {Size = UDim2.new(1, 0, 1, 0)}, 0.2)

      wait(0.2)
      utility:Tween(notification, {
         Size = UDim2.new(0, 0, 0, 60),
         Position = notification.Position + UDim2.new(0, textSize.X + 70, 0, 0)
      }, 0.2)

      wait(0.2)
      notification:Destroy()
   end

   self.activeNotification = close

   notification.Accept.MouseButton1Click:Connect(function()

      if not active then 
         return
      end

      if callback then
         callback(true)
      end

      close()
   end)

   notification.Decline.MouseButton1Click:Connect(function()

      if not active then 
         return
      end

      if callback then
         callback(false)
      end

      close()
   end)
end

function section:addButton(title, callback)
   local button = utility:Create("ImageButton", {
      Name = "Button",
      Parent = self.container,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Size = UDim2.new(1, 0, 0, 30),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.DarkContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298)
   }, {
      utility:Create("TextLabel", {
         Name = "Title",
         BackgroundTransparency = 1,
         Size = UDim2.new(1, 0, 1, 0),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.10000000149012
      })
   })

   table.insert(self.modules, button)
   --self:Resize()

   local text = button.Title
   local debounce

   button.MouseButton1Click:Connect(function()

      if debounce then
         return
      end

      -- animation
      utility:Pop(button, 10)

      debounce = true
      text.TextSize = 0
      utility:Tween(button.Title, {TextSize = 14}, 0.2)

      wait(0.2)
      utility:Tween(button.Title, {TextSize = 12}, 0.2)

      if callback then
         callback(function(...)
            self:updateButton(button, ...)
         end)
      end

      debounce = false
   end)

   return button
end

function section:addToggle(title, default, callback)
   local toggle = utility:Create("ImageButton", {
      Name = "Toggle",
      Parent = self.container,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Size = UDim2.new(1, 0, 0, 30),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.DarkContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298)
   },{
      utility:Create("TextLabel", {
         Name = "Title",
         AnchorPoint = Vector2.new(0, 0.5),
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0.5, 1),
         Size = UDim2.new(0.5, 0, 1, 0),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.10000000149012,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("ImageLabel", {
         Name = "Button",
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Position = UDim2.new(1, -50, 0.5, -8),
         Size = UDim2.new(0, 40, 0, 16),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = themes.LightContrast,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      }, {
         utility:Create("ImageLabel", {
            Name = "Frame",
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 2, 0.5, -6),
            Size = UDim2.new(1, -22, 1, -4),
            ZIndex = 2,
            Image = "rbxassetid://5028857472",
            ImageColor3 = themes.TextColor,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 298, 298)
         })
      })
   })

   table.insert(self.modules, toggle)
   --self:Resize()

   local active = default
   local active = default
   self:updateToggle(toggle, nil, active)

   toggle.MouseButton1Click:Connect(function()
      active = not active
      self:updateToggle(toggle, nil, active)

      if callback then
         callback(active, function(...)
            self:updateToggle(toggle, ...)
         end)
      end
   end)

   return toggle
end

function section:addTextbox(title, default, callback)
   local textbox = utility:Create("ImageButton", {
      Name = "Textbox",
      Parent = self.container,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Size = UDim2.new(1, 0, 0, 30),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.DarkContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298)
   }, {
      utility:Create("TextLabel", {
         Name = "Title",
         AnchorPoint = Vector2.new(0, 0.5),
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0.5, 1),
         Size = UDim2.new(0.5, 0, 1, 0),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.10000000149012,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("ImageLabel", {
         Name = "Button",
         BackgroundTransparency = 1,
         Position = UDim2.new(1, -110, 0.5, -8),
         Size = UDim2.new(0, 100, 0, 16),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = themes.LightContrast,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      }, {
         utility:Create("TextBox", {
            Name = "Textbox", 
            BackgroundTransparency = 1,
            TextTruncate = Enum.TextTruncate.AtEnd,
            Position = UDim2.new(0, 5, 0, 0),
            Size = UDim2.new(1, -10, 1, 0),
            ZIndex = 3,
            Font = Enum.Font.GothamSemibold,
            Text = default or "",
            TextColor3 = themes.TextColor,
            TextSize = 11
         })
      })
   })

   table.insert(self.modules, textbox)
   --self:Resize()

   local button = textbox.Button
   local input = button.Textbox

   textbox.MouseButton1Click:Connect(function()

      if textbox.Button.Size ~= UDim2.new(0, 100, 0, 16) then
         return
      end

      utility:Tween(textbox.Button, {
         Size = UDim2.new(0, 200, 0, 16),
         Position = UDim2.new(1, -210, 0.5, -8)
      }, 0.2)

      wait()

      input.TextXAlignment = Enum.TextXAlignment.Left
      input:CaptureFocus()
   end)

   input:GetPropertyChangedSignal("Text"):Connect(function()

      if button.ImageTransparency == 0 and (button.Size == UDim2.new(0, 200, 0, 16) or button.Size == UDim2.new(0, 100, 0, 16)) then -- i know, i dont like this either
         utility:Pop(button, 10)
      end

      if callback then
         callback(input.Text, nil, function(...)
            self:updateTextbox(textbox, ...)
         end)
      end
   end)

   input.FocusLost:Connect(function()

      input.TextXAlignment = Enum.TextXAlignment.Center

      utility:Tween(textbox.Button, {
         Size = UDim2.new(0, 100, 0, 16),
         Position = UDim2.new(1, -110, 0.5, -8)
      }, 0.2)

      if callback then
         callback(input.Text, true, function(...)
            self:updateTextbox(textbox, ...)
         end)
      end
   end)

   return textbox
end

function section:addKeybind(title, default, callback, changedCallback)
   local keybind = utility:Create("ImageButton", {
      Name = "Keybind",
      Parent = self.container,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Size = UDim2.new(1, 0, 0, 30),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.DarkContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298)
   }, {
      utility:Create("TextLabel", {
         Name = "Title",
         AnchorPoint = Vector2.new(0, 0.5),
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0.5, 1),
         Size = UDim2.new(1, 0, 1, 0),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.10000000149012,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("ImageLabel", {
         Name = "Button",
         BackgroundTransparency = 1,
         Position = UDim2.new(1, -110, 0.5, -8),
         Size = UDim2.new(0, 100, 0, 16),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = themes.LightContrast,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      }, {
         utility:Create("TextLabel", {
            Name = "Text",
            BackgroundTransparency = 1,
            ClipsDescendants = true,
            Size = UDim2.new(1, 0, 1, 0),
            ZIndex = 3,
            Font = Enum.Font.GothamSemibold,
            Text = default and default.Name or "None",
            TextColor3 = themes.TextColor,
            TextSize = 11
         })
      })
   })

   table.insert(self.modules, keybind)
   --self:Resize()

   local text = keybind.Button.Text
   local button = keybind.Button

   local animate = function()
      if button.ImageTransparency == 0 then
         utility:Pop(button, 10)
      end
   end

   self.binds[keybind] = {callback = function()
      animate()

      if callback then
         callback(function(...)
            self:updateKeybind(keybind, ...)
         end)
      end
   end}

   if default and callback then
      self:updateKeybind(keybind, nil, default)
   end

   keybind.MouseButton1Click:Connect(function()

      animate()

      if self.binds[keybind].connection then -- unbind
         return self:updateKeybind(keybind)
      end

      if text.Text == "None" then -- new bind
         text.Text = "..."

         local key = utility:KeyPressed()

         self:updateKeybind(keybind, nil, key.KeyCode)
         animate()

         if changedCallback then
            changedCallback(key, function(...)
               self:updateKeybind(keybind, ...)
            end)
         end
      end
   end)

   return keybind
end

function section:addColorPicker(title, default, callback)
   local colorpicker = utility:Create("ImageButton", {
      Name = "ColorPicker",
      Parent = self.container,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Size = UDim2.new(1, 0, 0, 30),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.DarkContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298)
   },{
      utility:Create("TextLabel", {
         Name = "Title",
         AnchorPoint = Vector2.new(0, 0.5),
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0.5, 1),
         Size = UDim2.new(0.5, 0, 1, 0),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.10000000149012,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("ImageButton", {
         Name = "Button",
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Position = UDim2.new(1, -50, 0.5, -7),
         Size = UDim2.new(0, 40, 0, 14),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = Color3.fromRGB(255, 255, 255),
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      })
   })

   local tab = utility:Create("ImageLabel", {
      Name = "ColorPicker",
      Parent = self.page.library.container,
      BackgroundTransparency = 1,
      Position = UDim2.new(0.75, 0, 0.400000006, 0),
      Selectable = true,
      AnchorPoint = Vector2.new(0.5, 0.5),
      Size = UDim2.new(0, 162, 0, 169),
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.Background,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298),
      Visible = false,
   }, {
      utility:Create("ImageLabel", {
         Name = "Glow",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, -15, 0, -15),
         Size = UDim2.new(1, 30, 1, 30),
         ZIndex = 0,
         Image = "rbxassetid://5028857084",
         ImageColor3 = themes.Glow,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(22, 22, 278, 278)
      }),
      utility:Create("TextLabel", {
         Name = "Title",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0, 8),
         Size = UDim2.new(1, -40, 0, 16),
         ZIndex = 2,
         Font = Enum.Font.GothamSemibold,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 14,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("ImageButton", {
         Name = "Close",
         BackgroundTransparency = 1,
         Position = UDim2.new(1, -26, 0, 8),
         Size = UDim2.new(0, 16, 0, 16),
         ZIndex = 2,
         Image = "rbxassetid://5012538583",
         ImageColor3 = themes.TextColor
      }), 
      utility:Create("Frame", {
         Name = "Container",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 8, 0, 32),
         Size = UDim2.new(1, -18, 1, -40)
      }, {
         utility:Create("UIListLayout", {
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 6)
         }),
         utility:Create("ImageButton", {
            Name = "Canvas",
            BackgroundTransparency = 1,
            BorderColor3 = themes.LightContrast,
            Size = UDim2.new(1, 0, 0, 60),
            AutoButtonColor = false,
            Image = "rbxassetid://5108535320",
            ImageColor3 = Color3.fromRGB(255, 0, 0),
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 298, 298)
         }, {
            utility:Create("ImageLabel", {
               Name = "White_Overlay",
               BackgroundTransparency = 1,
               Size = UDim2.new(1, 0, 0, 60),
               Image = "rbxassetid://5107152351",
               SliceCenter = Rect.new(2, 2, 298, 298)
            }),
            utility:Create("ImageLabel", {
               Name = "Black_Overlay",
               BackgroundTransparency = 1,
               Size = UDim2.new(1, 0, 0, 60),
               Image = "rbxassetid://5107152095",
               SliceCenter = Rect.new(2, 2, 298, 298)
            }),
            utility:Create("ImageLabel", {
               Name = "Cursor",
               BackgroundColor3 = themes.TextColor,
               AnchorPoint = Vector2.new(0.5, 0.5),
               BackgroundTransparency = 1.000,
               Size = UDim2.new(0, 10, 0, 10),
               Position = UDim2.new(0, 0, 0, 0),
               Image = "rbxassetid://5100115962",
               SliceCenter = Rect.new(2, 2, 298, 298)
            })
         }),
         utility:Create("ImageButton", {
            Name = "Color",
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Position = UDim2.new(0, 0, 0, 4),
            Selectable = false,
            Size = UDim2.new(1, 0, 0, 16),
            ZIndex = 2,
            AutoButtonColor = false,
            Image = "rbxassetid://5028857472",
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 298, 298)
         }, {
            utility:Create("Frame", {
               Name = "Select",
               BackgroundColor3 = themes.TextColor,
               BorderSizePixel = 1,
               Position = UDim2.new(1, 0, 0, 0),
               Size = UDim2.new(0, 2, 1, 0),
               ZIndex = 2
            }),
            utility:Create("UIGradient", { -- rainbow canvas
               Color = ColorSequence.new({
                  ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 0, 0)), 
                  ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 255, 0)), 
                  ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 255, 0)), 
                  ColorSequenceKeypoint.new(0.50, Color3.fromRGB(0, 255, 255)), 
                  ColorSequenceKeypoint.new(0.66, Color3.fromRGB(0, 0, 255)), 
                  ColorSequenceKeypoint.new(0.82, Color3.fromRGB(255, 0, 255)), 
                  ColorSequenceKeypoint.new(1.00, Color3.fromRGB(255, 0, 0))
               })
            })
         }),
         utility:Create("Frame", {
            Name = "Inputs",
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 158),
            Size = UDim2.new(1, 0, 0, 16)
         }, {
            utility:Create("UIListLayout", {
               FillDirection = Enum.FillDirection.Horizontal,
               SortOrder = Enum.SortOrder.LayoutOrder,
               Padding = UDim.new(0, 6)
            }),
            utility:Create("ImageLabel", {
               Name = "R",
               BackgroundTransparency = 1,
               BorderSizePixel = 0,
               Size = UDim2.new(0.305, 0, 1, 0),
               ZIndex = 2,
               Image = "rbxassetid://5028857472",
               ImageColor3 = themes.DarkContrast,
               ScaleType = Enum.ScaleType.Slice,
               SliceCenter = Rect.new(2, 2, 298, 298)
            }, {
               utility:Create("TextLabel", {
                  Name = "Text",
                  BackgroundTransparency = 1,
                  Size = UDim2.new(0.400000006, 0, 1, 0),
                  ZIndex = 2,
                  Font = Enum.Font.Gotham,
                  Text = "R:",
                  TextColor3 = themes.TextColor,
                  TextSize = 10.000
               }),
               utility:Create("TextBox", {
                  Name = "Textbox",
                  BackgroundTransparency = 1,
                  Position = UDim2.new(0.300000012, 0, 0, 0),
                  Size = UDim2.new(0.600000024, 0, 1, 0),
                  ZIndex = 2,
                  Font = Enum.Font.Gotham,
                  PlaceholderColor3 = themes.DarkContrast,
                  Text = "255",
                  TextColor3 = themes.TextColor,
                  TextSize = 10.000
               })
            }),
            utility:Create("ImageLabel", {
               Name = "G",
               BackgroundTransparency = 1,
               BorderSizePixel = 0,
               Size = UDim2.new(0.305, 0, 1, 0),
               ZIndex = 2,
               Image = "rbxassetid://5028857472",
               ImageColor3 = themes.DarkContrast,
               ScaleType = Enum.ScaleType.Slice,
               SliceCenter = Rect.new(2, 2, 298, 298)
            }, {
               utility:Create("TextLabel", {
                  Name = "Text",
                  BackgroundTransparency = 1,
                  ZIndex = 2,
                  Size = UDim2.new(0.400000006, 0, 1, 0),
                  Font = Enum.Font.Gotham,
                  Text = "G:",
                  TextColor3 = themes.TextColor,
                  TextSize = 10.000
               }),
               utility:Create("TextBox", {
                  Name = "Textbox",
                  BackgroundTransparency = 1,
                  Position = UDim2.new(0.300000012, 0, 0, 0),
                  Size = UDim2.new(0.600000024, 0, 1, 0),
                  ZIndex = 2,
                  Font = Enum.Font.Gotham,
                  Text = "255",
                  TextColor3 = themes.TextColor,
                  TextSize = 10.000
               })
            }),
            utility:Create("ImageLabel", {
               Name = "B",
               BackgroundTransparency = 1,
               BorderSizePixel = 0,
               Size = UDim2.new(0.305, 0, 1, 0),
               ZIndex = 2,
               Image = "rbxassetid://5028857472",
               ImageColor3 = themes.DarkContrast,
               ScaleType = Enum.ScaleType.Slice,
               SliceCenter = Rect.new(2, 2, 298, 298)
            }, {
               utility:Create("TextLabel", {
                  Name = "Text",
                  BackgroundTransparency = 1,
                  Size = UDim2.new(0.400000006, 0, 1, 0),
                  ZIndex = 2,
                  Font = Enum.Font.Gotham,
                  Text = "B:",
                  TextColor3 = themes.TextColor,
                  TextSize = 10.000
               }),
               utility:Create("TextBox", {
                  Name = "Textbox",
                  BackgroundTransparency = 1,
                  Position = UDim2.new(0.300000012, 0, 0, 0),
                  Size = UDim2.new(0.600000024, 0, 1, 0),
                  ZIndex = 2,
                  Font = Enum.Font.Gotham,
                  Text = "255",
                  TextColor3 = themes.TextColor,
                  TextSize = 10.000
               })
            }),
         }),
         utility:Create("ImageButton", {
            Name = "Button",
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 0, 20),
            ZIndex = 2,
            Image = "rbxassetid://5028857472",
            ImageColor3 = themes.DarkContrast,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 298, 298)
         }, {
            utility:Create("TextLabel", {
               Name = "Text",
               BackgroundTransparency = 1,
               Size = UDim2.new(1, 0, 1, 0),
               ZIndex = 3,
               Font = Enum.Font.Gotham,
               Text = "Submit",
               TextColor3 = themes.TextColor,
               TextSize = 11.000
            })
         })
      })
   })

   utility:DraggingEnabled(tab)
   table.insert(self.modules, colorpicker)
   --self:Resize()

   local allowed = {
      [""] = true
   }

   local canvas = tab.Container.Canvas
   local color = tab.Container.Color

   local canvasSize, canvasPosition = canvas.AbsoluteSize, canvas.AbsolutePosition
   local colorSize, colorPosition = color.AbsoluteSize, color.AbsolutePosition

   local draggingColor, draggingCanvas

   local color3 = default or Color3.fromRGB(255, 255, 255)
   local hue, sat, brightness = 0, 0, 1
   local rgb = {
      r = 255,
      g = 255,
      b = 255
   }

   self.colorpickers[colorpicker] = {
      tab = tab,
      callback = function(prop, value)
         rgb[prop] = value
         hue, sat, brightness = Color3.toHSV(Color3.fromRGB(rgb.r, rgb.g, rgb.b))
      end
   }

   local callback = function(value)
      if callback then
         callback(value, function(...)
            self:updateColorPicker(colorpicker, ...)
         end)
      end
   end

   utility:DraggingEnded(function()
      draggingColor, draggingCanvas = false, false
   end)

   if default then
      self:updateColorPicker(colorpicker, nil, default)

      hue, sat, brightness = Color3.toHSV(default)
      default = Color3.fromHSV(hue, sat, brightness)

      for i, prop in pairs({"r", "g", "b"}) do
         rgb[prop] = default[prop:upper()] * 255
      end
   end

   for i, container in pairs(tab.Container.Inputs:GetChildren()) do -- i know what you are about to say, so shut up
      if container:IsA("ImageLabel") then
         local textbox = container.Textbox
         local focused

         textbox.Focused:Connect(function()
            focused = true
         end)

         textbox.FocusLost:Connect(function()
            focused = false

            if not tonumber(textbox.Text) then
               textbox.Text = math.floor(rgb[container.Name:lower()])
            end
         end)

         textbox:GetPropertyChangedSignal("Text"):Connect(function()
            local text = textbox.Text

            if not allowed[text] and not tonumber(text) then
               textbox.Text = text:sub(1, #text - 1)
            elseif focused and not allowed[text] then
               rgb[container.Name:lower()] = math.clamp(tonumber(textbox.Text), 0, 255)

               local color3 = Color3.fromRGB(rgb.r, rgb.g, rgb.b)
               hue, sat, brightness = Color3.toHSV(color3)

               self:updateColorPicker(colorpicker, nil, color3)
               callback(color3)
            end
         end)
      end
   end

   canvas.MouseButton1Down:Connect(function()
      draggingCanvas = true

      while draggingCanvas do

         local x, y = mouse.X, mouse.Y

         sat = math.clamp((x - canvasPosition.X) / canvasSize.X, 0, 1)
         brightness = 1 - math.clamp((y - canvasPosition.Y) / canvasSize.Y, 0, 1)

         color3 = Color3.fromHSV(hue, sat, brightness)

         for i, prop in pairs({"r", "g", "b"}) do
            rgb[prop] = color3[prop:upper()] * 255
         end

         self:updateColorPicker(colorpicker, nil, {hue, sat, brightness}) -- roblox is literally retarded
         utility:Tween(canvas.Cursor, {Position = UDim2.new(sat, 0, 1 - brightness, 0)}, 0.1) -- overwrite

         callback(color3)
         utility:Wait()
      end
   end)

   color.MouseButton1Down:Connect(function()
      draggingColor = true

      while draggingColor do

         hue = 1 - math.clamp(1 - ((mouse.X - colorPosition.X) / colorSize.X), 0, 1)
         color3 = Color3.fromHSV(hue, sat, brightness)

         for i, prop in pairs({"r", "g", "b"}) do
            rgb[prop] = color3[prop:upper()] * 255
         end

         local x = hue -- hue is updated
         self:updateColorPicker(colorpicker, nil, {hue, sat, brightness}) -- roblox is literally retarded
         utility:Tween(tab.Container.Color.Select, {Position = UDim2.new(x, 0, 0, 0)}, 0.1) -- overwrite

         callback(color3)
         utility:Wait()
      end
   end)

   -- click events
   local button = colorpicker.Button
   local toggle, debounce, animate

   lastColor = Color3.fromHSV(hue, sat, brightness)
   animate = function(visible, overwrite)

      if overwrite then

         if not toggle then
            return
         end

         if debounce then
            while debounce do
               utility:Wait()
            end
         end
      elseif not overwrite then
         if debounce then 
            return 
         end

         if button.ImageTransparency == 0 then
            utility:Pop(button, 10)
         end
      end

      toggle = visible
      debounce = true

      if visible then

         if self.page.library.activePicker and self.page.library.activePicker ~= animate then
            self.page.library.activePicker(nil, true)
         end

         self.page.library.activePicker = animate
         lastColor = Color3.fromHSV(hue, sat, brightness)

         local x1, x2 = button.AbsoluteSize.X / 2, 162--tab.AbsoluteSize.X
         local px, py = button.AbsolutePosition.X, button.AbsolutePosition.Y

         tab.ClipsDescendants = true
         tab.Visible = true
         tab.Size = UDim2.new(0, 0, 0, 0)

         tab.Position = UDim2.new(0, x1 + x2 + px, 0, py)
         utility:Tween(tab, {Size = UDim2.new(0, 162, 0, 169)}, 0.2)

         -- update size and position
         wait(0.2)
         tab.ClipsDescendants = false

         canvasSize, canvasPosition = canvas.AbsoluteSize, canvas.AbsolutePosition
         colorSize, colorPosition = color.AbsoluteSize, color.AbsolutePosition
      else
         utility:Tween(tab, {Size = UDim2.new(0, 0, 0, 0)}, 0.2)
         tab.ClipsDescendants = true

         wait(0.2)
         tab.Visible = false
      end

      debounce = false
   end

   local toggleTab = function()
      animate(not toggle)
   end

   button.MouseButton1Click:Connect(toggleTab)
   colorpicker.MouseButton1Click:Connect(toggleTab)

   tab.Container.Button.MouseButton1Click:Connect(function()
      animate()
   end)

   tab.Close.MouseButton1Click:Connect(function()
      self:updateColorPicker(colorpicker, nil, lastColor)
      animate()
   end)

   return colorpicker
end

function section:addSlider(title, default, min, max, callback)
   local slider = utility:Create("ImageButton", {
      Name = "Slider",
      Parent = self.container,
      BackgroundTransparency = 1,
      BorderSizePixel = 0,
      Position = UDim2.new(0.292817682, 0, 0.299145311, 0),
      Size = UDim2.new(1, 0, 0, 50),
      ZIndex = 2,
      Image = "rbxassetid://5028857472",
      ImageColor3 = themes.DarkContrast,
      ScaleType = Enum.ScaleType.Slice,
      SliceCenter = Rect.new(2, 2, 298, 298)
   }, {
      utility:Create("TextLabel", {
         Name = "Title",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0, 6),
         Size = UDim2.new(0.5, 0, 0, 16),
         ZIndex = 3,
         Font = Enum.Font.Gotham,
         Text = title,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextTransparency = 0.10000000149012,
         TextXAlignment = Enum.TextXAlignment.Left
      }),
      utility:Create("TextBox", {
         Name = "TextBox",
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Position = UDim2.new(1, -30, 0, 6),
         Size = UDim2.new(0, 20, 0, 16),
         ZIndex = 3,
         Font = Enum.Font.GothamSemibold,
         Text = default or min,
         TextColor3 = themes.TextColor,
         TextSize = 12,
         TextXAlignment = Enum.TextXAlignment.Right
      }),
      utility:Create("TextLabel", {
         Name = "Slider",
         BackgroundTransparency = 1,
         Position = UDim2.new(0, 10, 0, 28),
         Size = UDim2.new(1, -20, 0, 16),
         ZIndex = 3,
         Text = "",
      }, {
         utility:Create("ImageLabel", {
            Name = "Bar",
            AnchorPoint = Vector2.new(0, 0.5),
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 0, 0.5, 0),
            Size = UDim2.new(1, 0, 0, 4),
            ZIndex = 3,
            Image = "rbxassetid://5028857472",
            ImageColor3 = themes.LightContrast,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(2, 2, 298, 298)
         }, {
            utility:Create("ImageLabel", {
               Name = "Fill",
               BackgroundTransparency = 1,
               Size = UDim2.new(0.8, 0, 1, 0),
               ZIndex = 3,
               Image = "rbxassetid://5028857472",
               ImageColor3 = themes.TextColor,
               ScaleType = Enum.ScaleType.Slice,
               SliceCenter = Rect.new(2, 2, 298, 298)
            }, {
               utility:Create("ImageLabel", {
                  Name = "Circle",
                  AnchorPoint = Vector2.new(0.5, 0.5),
                  BackgroundTransparency = 1,
                  ImageTransparency = 1.000,
                  ImageColor3 = themes.TextColor,
                  Position = UDim2.new(1, 0, 0.5, 0),
                  Size = UDim2.new(0, 10, 0, 10),
                  ZIndex = 3,
                  Image = "rbxassetid://4608020054"
               })
            })
         })
      })
   })

   table.insert(self.modules, slider)
   --self:Resize()

   local allowed = {
      [""] = true,
      ["-"] = true
   }

   local textbox = slider.TextBox
   local circle = slider.Slider.Bar.Fill.Circle

   local value = default or min
   local dragging, last

   local callback = function(value)
      if callback then
         callback(value, function(...)
            self:updateSlider(slider, ...)
         end)
      end
   end

   self:updateSlider(slider, nil, value, min, max)

   utility:DraggingEnded(function()
      dragging = false
   end)

   slider.MouseButton1Down:Connect(function(input)
      dragging = true

      while dragging do
         utility:Tween(circle, {ImageTransparency = 0}, 0.1)

         value = self:updateSlider(slider, nil, nil, min, max, value)
         callback(value)

         utility:Wait()
      end

      wait(0.5)
      utility:Tween(circle, {ImageTransparency = 1}, 0.2)
   end)

   textbox.FocusLost:Connect(function()
      if not tonumber(textbox.Text) then
         value = self:updateSlider(slider, nil, default or min, min, max)
         callback(value)
      end
   end)

   textbox:GetPropertyChangedSignal("Text"):Connect(function()
      local text = textbox.Text

      if not allowed[text] and not tonumber(text) then
         textbox.Text = text:sub(1, #text - 1)
      elseif not allowed[text] then	
         value = self:updateSlider(slider, nil, tonumber(text) or value, min, max)
         callback(value)
      end
   end)

   return slider
end

function section:addDropdown(title, list, callback)
   local dropdown = utility:Create("Frame", {
      Name = "Dropdown",
      Parent = self.container,
      BackgroundTransparency = 1,
      Size = UDim2.new(1, 0, 0, 30),
      ClipsDescendants = true
   }, {
      utility:Create("UIListLayout", {
         SortOrder = Enum.SortOrder.LayoutOrder,
         Padding = UDim.new(0, 4)
      }),
      utility:Create("ImageLabel", {
         Name = "Search",
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Size = UDim2.new(1, 0, 0, 30),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = themes.DarkContrast,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      }, {
         utility:Create("TextBox", {
            Name = "TextBox",
            AnchorPoint = Vector2.new(0, 0.5),
            BackgroundTransparency = 1,
            TextTruncate = Enum.TextTruncate.AtEnd,
            Position = UDim2.new(0, 10, 0.5, 1),
            Size = UDim2.new(1, -42, 1, 0),
            ZIndex = 3,
            Font = Enum.Font.Gotham,
            Text = title,
            TextColor3 = themes.TextColor,
            TextSize = 12,
            TextTransparency = 0.10000000149012,
            TextXAlignment = Enum.TextXAlignment.Left
         }),
         utility:Create("ImageButton", {
            Name = "Button",
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Position = UDim2.new(1, -28, 0.5, -9),
            Size = UDim2.new(0, 18, 0, 18),
            ZIndex = 3,
            Image = "rbxassetid://5012539403",
            ImageColor3 = themes.TextColor,
            SliceCenter = Rect.new(2, 2, 298, 298)
         })
      }),
      utility:Create("ImageLabel", {
         Name = "List",
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Size = UDim2.new(1, 0, 1, -34),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = themes.Background,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      }, {
         utility:Create("ScrollingFrame", {
            Name = "Frame",
            Active = true,
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Position = UDim2.new(0, 4, 0, 4),
            Size = UDim2.new(1, -8, 1, -8),
            CanvasPosition = Vector2.new(0, 28),
            CanvasSize = UDim2.new(0, 0, 0, 120),
            ZIndex = 2,
            ScrollBarThickness = 3,
            ScrollBarImageColor3 = themes.DarkContrast
         }, {
            utility:Create("UIListLayout", {
               SortOrder = Enum.SortOrder.LayoutOrder,
               Padding = UDim.new(0, 4)
            })
         })
      })
   })

   table.insert(self.modules, dropdown)
   --self:Resize()

   local search = dropdown.Search
   local focused

   list = list or {}

   search.Button.MouseButton1Click:Connect(function()
      if search.Button.Rotation == 0 then
         self:updateDropdown(dropdown, nil, list, callback)
      else
         self:updateDropdown(dropdown, nil, nil, callback)
      end
   end)

   search.TextBox.Focused:Connect(function()
      if search.Button.Rotation == 0 then
         self:updateDropdown(dropdown, nil, list, callback)
      end

      focused = true
   end)

   search.TextBox.FocusLost:Connect(function()
      focused = false
   end)

   search.TextBox:GetPropertyChangedSignal("Text"):Connect(function()
      if focused then
         local list = utility:Sort(search.TextBox.Text, list)
         list = #list ~= 0 and list 

         self:updateDropdown(dropdown, nil, list, callback)
      end
   end)

   dropdown:GetPropertyChangedSignal("Size"):Connect(function()
      self:Resize()
   end)

   return dropdown
end

-- class functions

function library:SelectPage(page, toggle)

   if toggle and self.focusedPage == page then -- already selected
      return
   end

   local button = page.button

   if toggle then
      -- page button
      button.Title.TextTransparency = 0
      button.Title.Font = Enum.Font.GothamSemibold

      if button:FindFirstChild("Icon") then
         button.Icon.ImageTransparency = 0
      end

      -- update selected page
      local focusedPage = self.focusedPage
      self.focusedPage = page

      if focusedPage then
         self:SelectPage(focusedPage)
      end

      -- sections
      local existingSections = focusedPage and #focusedPage.sections or 0
      local sectionsRequired = #page.sections - existingSections

      page:Resize()

      for i, section in pairs(page.sections) do
         section.container.Parent.ImageTransparency = 0
      end

      if sectionsRequired < 0 then -- "hides" some sections
         for i = existingSections, #page.sections + 1, -1 do
            local section = focusedPage.sections[i].container.Parent

            utility:Tween(section, {ImageTransparency = 1}, 0.1)
         end
      end

      wait(0.1)
      page.container.Visible = true

      if focusedPage then
         focusedPage.container.Visible = false
      end

      if sectionsRequired > 0 then -- "creates" more section
         for i = existingSections + 1, #page.sections do
            local section = page.sections[i].container.Parent

            section.ImageTransparency = 1
            utility:Tween(section, {ImageTransparency = 0}, 0.05)
         end
      end

      wait(0.05)

      for i, section in pairs(page.sections) do

         utility:Tween(section.container.Title, {TextTransparency = 0}, 0.1)
         section:Resize(true)

         wait(0.05)
      end

      wait(0.05)
      page:Resize(true)
   else
      -- page button
      button.Title.Font = Enum.Font.Gotham
      button.Title.TextTransparency = 0.65

      if button:FindFirstChild("Icon") then
         button.Icon.ImageTransparency = 0.65
      end

      -- sections
      for i, section in pairs(page.sections) do	
         utility:Tween(section.container.Parent, {Size = UDim2.new(1, -10, 0, 28)}, 0.1)
         utility:Tween(section.container.Title, {TextTransparency = 1}, 0.1)
      end

      wait(0.1)

      page.lastPosition = page.container.CanvasPosition.Y
      page:Resize()
   end
end

function page:Resize(scroll)
   local padding = 10
   local size = 0

   for i, section in pairs(self.sections) do
      size = size + section.container.Parent.AbsoluteSize.Y + padding
   end

   self.container.CanvasSize = UDim2.new(0, 0, 0, size)
   self.container.ScrollBarImageTransparency = size > self.container.AbsoluteSize.Y

   if scroll then
      utility:Tween(self.container, {CanvasPosition = Vector2.new(0, self.lastPosition or 0)}, 0.2)
   end
end

function section:Resize(smooth)

   if self.page.library.focusedPage ~= self.page then
      return
   end

   local padding = 4
   local size = (4 * padding) + self.container.Title.AbsoluteSize.Y -- offset

   for i, module in pairs(self.modules) do
      size = size + module.AbsoluteSize.Y + padding
   end

   if smooth then
      utility:Tween(self.container.Parent, {Size = UDim2.new(1, -10, 0, size)}, 0.05)
   else
      self.container.Parent.Size = UDim2.new(1, -10, 0, size)
      self.page:Resize()
   end
end

function section:getModule(info)

   if table.find(self.modules, info) then
      return info
   end

   for i, module in pairs(self.modules) do
      if (module:FindFirstChild("Title") or module:FindFirstChild("TextBox", true)).Text == info then
         return module
      end
   end

   error("No module found under "..tostring(info))
end

-- updates

function section:updateButton(button, title)
   button = self:getModule(button)

   button.Title.Text = title
end

function section:updateToggle(toggle, title, value)
   toggle = self:getModule(toggle)

   local position = {
      In = UDim2.new(0, 2, 0.5, -6),
      Out = UDim2.new(0, 20, 0.5, -6)
   }

   local frame = toggle.Button.Frame
   value = value and "Out" or "In"

   if title then
      toggle.Title.Text = title
   end

   utility:Tween(frame, {
      Size = UDim2.new(1, -22, 1, -9),
      Position = position[value] + UDim2.new(0, 0, 0, 2.5)
   }, 0.2)

   wait(0.1)
   utility:Tween(frame, {
      Size = UDim2.new(1, -22, 1, -4),
      Position = position[value]
   }, 0.1)
end

function section:updateTextbox(textbox, title, value)
   textbox = self:getModule(textbox)

   if title then
      textbox.Title.Text = title
   end

   if value then
      textbox.Button.Textbox.Text = value
   end

end

function section:updateKeybind(keybind, title, key)
   keybind = self:getModule(keybind)

   local text = keybind.Button.Text
   local bind = self.binds[keybind]

   if title then
      keybind.Title.Text = title
   end

   if bind.connection then
      bind.connection = bind.connection:UnBind()
   end

   if key then
      self.binds[keybind].connection = utility:BindToKey(key, bind.callback)
      text.Text = key.Name
   else
      text.Text = "None"
   end
end

function section:updateColorPicker(colorpicker, title, color)
   colorpicker = self:getModule(colorpicker)

   local picker = self.colorpickers[colorpicker]
   local tab = picker.tab
   local callback = picker.callback

   if title then
      colorpicker.Title.Text = title
      tab.Title.Text = title
   end

   local color3
   local hue, sat, brightness

   if type(color) == "table" then -- roblox is literally retarded x2
      hue, sat, brightness = unpack(color)
      color3 = Color3.fromHSV(hue, sat, brightness)
   else
      color3 = color
      hue, sat, brightness = Color3.toHSV(color3)
   end

   utility:Tween(colorpicker.Button, {ImageColor3 = color3}, 0.5)
   utility:Tween(tab.Container.Color.Select, {Position = UDim2.new(hue, 0, 0, 0)}, 0.1)

   utility:Tween(tab.Container.Canvas, {ImageColor3 = Color3.fromHSV(hue, 1, 1)}, 0.5)
   utility:Tween(tab.Container.Canvas.Cursor, {Position = UDim2.new(sat, 0, 1 - brightness)}, 0.5)

   for i, container in pairs(tab.Container.Inputs:GetChildren()) do
      if container:IsA("ImageLabel") then
         local value = math.clamp(color3[container.Name], 0, 1) * 255

         container.Textbox.Text = math.floor(value)
         --callback(container.Name:lower(), value)
      end
   end
end

function section:updateSlider(slider, title, value, min, max, lvalue)
   slider = self:getModule(slider)

   if title then
      slider.Title.Text = title
   end

   local bar = slider.Slider.Bar
   local percent = (mouse.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X

   if value then -- support negative ranges
      percent = (value - min) / (max - min)
   end

   percent = math.clamp(percent, 0, 1)
   value = value or math.floor(min + (max - min) * percent)

   slider.TextBox.Text = value
   utility:Tween(bar.Fill, {Size = UDim2.new(percent, 0, 1, 0)}, 0.1)

   if value ~= lvalue and slider.ImageTransparency == 0 then
      utility:Pop(slider, 10)
   end

   return value
end

function section:updateDropdown(dropdown, title, list, callback)
   dropdown = self:getModule(dropdown)

   if title then
      dropdown.Search.TextBox.Text = title
   end

   local entries = 0

   utility:Pop(dropdown.Search, 10)

   for i, button in pairs(dropdown.List.Frame:GetChildren()) do
      if button:IsA("ImageButton") then
         button:Destroy()
      end
   end

   for i, value in pairs(list or {}) do
      local button = utility:Create("ImageButton", {
         Parent = dropdown.List.Frame,
         BackgroundTransparency = 1,
         BorderSizePixel = 0,
         Size = UDim2.new(1, 0, 0, 30),
         ZIndex = 2,
         Image = "rbxassetid://5028857472",
         ImageColor3 = themes.DarkContrast,
         ScaleType = Enum.ScaleType.Slice,
         SliceCenter = Rect.new(2, 2, 298, 298)
      }, {
         utility:Create("TextLabel", {
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 0),
            Size = UDim2.new(1, -10, 1, 0),
            ZIndex = 3,
            Font = Enum.Font.Gotham,
            Text = value,
            TextColor3 = themes.TextColor,
            TextSize = 12,
            TextXAlignment = "Left",
            TextTransparency = 0.10000000149012
         })
      })

      button.MouseButton1Click:Connect(function()
         if callback then
            callback(value, function(...)
               self:updateDropdown(dropdown, ...)
            end)	
         end

         self:updateDropdown(dropdown, value, nil, callback)
      end)

      entries = entries + 1
   end

   local frame = dropdown.List.Frame

   utility:Tween(dropdown, {Size = UDim2.new(1, 0, 0, (entries == 0 and 30) or math.clamp(entries, 0, 3) * 34 + 38)}, 0.3)
   utility:Tween(dropdown.Search.Button, {Rotation = list and 180 or 0}, 0.3)

   if entries > 3 then

      for i, button in pairs(dropdown.List.Frame:GetChildren()) do
         if button:IsA("ImageButton") then
            button.Size = UDim2.new(1, -6, 0, 30)
         end
      end

      frame.CanvasSize = UDim2.new(0, 0, 0, (entries * 34) - 4)
      frame.ScrollBarImageTransparency = 0
   else
      frame.CanvasSize = UDim2.new(0, 0, 0, 0)
      frame.ScrollBarImageTransparency = 1
   end
end
end
FirstWorld = false
SecondWorld = false
ThirdWorld = false
if placeId == 2753915549 then
   FirstWorld = true
elseif placeId == 4442272183 then
   SecondWorld = true
elseif placeId == 7449423635 then
   ThirdWorld = true
end
--Check player stats--
local MyLevel = game.Players.localPlayer.Data.Level.Value
local MyFragment = game.Players.localPlayer.Data.Fragments.Value
local placeId = game.PlaceId
local MyBeli = game.Players.localPlayer.Data.Beli.Value
local MyDevilFruit = game.Players.localPlayer.Data.DevilFruit.Value

function zeroGrav(part)
   if part:FindFirstChild("BodyForce") then return end
   local temp = Instance.new("BodyForce")
   temp.Force = part:GetMass() * Vector3.new(0,workspace.Gravity,0)
   temp.Parent = part
end

function isnil(thing)
   return (thing == nil)
end

function round(n)
   return math.floor(tonumber(n) + 0.5)
end
 -- CheckQuest

function CheckQuest()
    local MyLevel = game.Players.localPlayer.Data.Level.Value
    if FirstWorld then
      if MyLevel == 1 or MyLevel <= 9 then -- Bandit
         Ms = "Bandit [Lv. 5]"
         NaemQuest = "BanditQuest1"
         LevelQuest = 1
         NameMon = "Bandit"
         CFrameQuest = CFrame.new(1061.66699, 16.5166187, 1544.52905, -0.942978859, -3.33851502e-09, 0.332852632, 7.04340497e-09, 1, 2.99841325e-08, -0.332852632, 3.06188177e-08, -0.942978859)
         CFrameMon = CFrame.new(1124.98352, 16.3941536, 1545.57556)
      elseif MyLevel == 10 or MyLevel <= 14 then -- Monkey
         Ms = "Monkey [Lv. 14]"
         NaemQuest = "JungleQuest"
         LevelQuest = 1
         NameMon = "Monkey"
         CFrameQuest = CFrame.new(-1604.12012, 36.8521118, 154.23732, 0.0648873374, -4.70858913e-06, -0.997892559, 1.41431883e-07, 1, -4.70933674e-06, 0.997892559, 1.64442184e-07, 0.0648873374)
         CFrameMon = CFrame.new(-1609.50916, 36.8520927, 146.766647)
      elseif MyLevel == 15 or MyLevel <= 29 then -- Gorilla
         Ms = "Gorilla [Lv. 20]"
         NaemQuest = "JungleQuest"
         LevelQuest = 2
         NameMon = "Gorilla"
         CFrameQuest = CFrame.new(-1604.12012, 36.8521118, 154.23732, 0.0648873374, -4.70858913e-06, -0.997892559, 1.41431883e-07, 1, -4.70933674e-06, 0.997892559, 1.64442184e-07, 0.0648873374)
         CFrameMon = CFrame.new(-1198.01721, 7.7187376, -448.22168)
        elseif MyLevel == 30 or MyLevel <= 39 then -- Pirate
         Ms = "Pirate [Lv. 35]"
         NaemQuest = "BuggyQuest1"
         LevelQuest = 1
         NameMon = "Pirate"
         CFrameQuest = CFrame.new(-1139.59717, 4.75205183, 3825.16211, -0.959730506, -7.5857054e-09, 0.280922383, -4.06310328e-08, 1, -1.11807175e-07, -0.280922383, -1.18718916e-07, -0.959730506)
         CFrameMon = CFrame.new(-1107.5354, 13.7520456, 3969.72046)
        elseif MyLevel == 40 or MyLevel <= 59 then -- Brute
         Ms = "Brute [Lv. 45]"
         NaemQuest = "BuggyQuest1"
         LevelQuest = 2
         NameMon = "Brute"
         CFrameQuest = CFrame.new(-1139.59717, 4.75205183, 3825.16211, -0.959730506, -7.5857054e-09, 0.280922383, -4.06310328e-08, 1, -1.11807175e-07, -0.280922383, -1.18718916e-07, -0.959730506)
         CFrameMon = CFrame.new(-1116.89844, 14.8098745, 4279.84961)
        elseif MyLevel == 60 or MyLevel <= 74 then -- Desert Bandit
         Ms = "Desert Bandit [Lv. 60]"
         NaemQuest = "DesertQuest"
         LevelQuest = 1
         NameMon = "Desert Bandit"
         CFrameQuest = CFrame.new(897.031128, 6.43846416, 4388.97168, -0.804044724, 3.68233266e-08, 0.594568789, 6.97835176e-08, 1, 3.24365246e-08, -0.594568789, 6.75715199e-08, -0.804044724)
         CFrameMon = CFrame.new(918.105469, 6.43846178, 4370.0249)
        elseif MyLevel == 75 or MyLevel <= 89 then -- Desert Officre
         Ms = "Desert Officer [Lv. 70]"
         NaemQuest = "DesertQuest"
         LevelQuest = 2
         NameMon = "Desert Officer"
         CFrameQuest = CFrame.new(897.031128, 6.43846416, 4388.97168, -0.804044724, 3.68233266e-08, 0.594568789, 6.97835176e-08, 1, 3.24365246e-08, -0.594568789, 6.75715199e-08, -0.804044724)
         CFrameMon = CFrame.new(1537.75635, 14.4520388, 4378.12891)
        elseif MyLevel == 90 or MyLevel <= 99 then -- Snow Bandits
           Ms = "Snow Bandit [Lv. 90]"
           NaemQuest = "SnowQuest"
           LevelQuest = 1
           NameMon = "Snow Bandits"
           CFrameQuest = CFrame.new(1384.14001, 87.272789, -1297.06482, 0.348555952, -2.53947841e-09, -0.937287986, 1.49860568e-08, 1, 2.86358204e-09, 0.937287986, -1.50443711e-08, 0.348555952)
           CFrameMon = CFrame.new(1389.47705, 87.272789, -1360.79626)
        elseif MyLevel == 100 or MyLevel <= 119 then -- Snowman
           Ms = "Snowman [Lv. 100]"
           NaemQuest = "SnowQuest"
           LevelQuest = 2
           NameMon = "Snowman"
           CFrameQuest = CFrame.new(1384.14001, 87.272789, -1297.06482, 0.348555952, -2.53947841e-09, -0.937287986, 1.49860568e-08, 1, 2.86358204e-09, 0.937287986, -1.50443711e-08, 0.348555952)
           CFrameMon = CFrame.new(1234.13159, 105.777832, -1417.06055)
        elseif MyLevel == 120 or MyLevel <= 149 then -- Chief Petty Officer
           Ms = "Chief Petty Officer [Lv. 120]"
           NaemQuest = "MarineQuest2"
           LevelQuest = 1
           NameMon = "Chief Petty Officer"
           CFrameQuest = CFrame.new(-5035.0835, 28.6520386, 4325.29443, 0.0243340395, -7.08064647e-08, 0.999703884, -6.36926814e-08, 1, 7.23777944e-08, -0.999703884, -6.54350671e-08, 0.0243340395)
           CFrameMon = CFrame.new(-4872.00049, 22.6520348, 4274.36523)
        elseif MyLevel == 150 or MyLevel <= 174 then -- Sky Bandit
           Ms = "Sky Bandit [Lv. 150]"
           NaemQuest = "SkyQuest"
           LevelQuest = 1
           NameMon = "Sky Bandit"
           CFrameQuest = CFrame.new(-4841.83447, 717.669617, -2623.96436, -0.875942111, 5.59710216e-08, -0.482416272, 3.04023082e-08, 1, 6.08195947e-08, 0.482416272, 3.86078725e-08, -0.875942111)
           CFrameMon = CFrame.new(-4955.36279, 295.744293, -2894.04858)
        elseif MyLevel == 175 or MyLevel <= 224 then -- Dark Master
           Ms = "Dark Master [Lv. 175]"
           NaemQuest = "SkyQuest"
           LevelQuest = 2
           NameMon = "Dark Master"
           CFrameQuest = CFrame.new(-4841.83447, 717.669617, -2623.96436, -0.875942111, 5.59710216e-08, -0.482416272, 3.04023082e-08, 1, 6.08195947e-08, 0.482416272, 3.86078725e-08, -0.875942111)
           CFrameMon = CFrame.new(-5247.03955, 393.063049, -2206.23926)
        elseif MyLevel == 225 or MyLevel <= 274 then -- Toga Warrior
           Ms = "Toga Warrior [Lv. 225]"
           NaemQuest = "ColosseumQuest"
           LevelQuest = 1
           NameMon = "Toga Warrior"
           CFrameQuest = CFrame.new(-1576.11743, 7.38933945, -2983.30762, 0.576966345, 1.22114863e-09, 0.816767931, -3.58496594e-10, 1, -1.24185606e-09, -0.816767931, 4.2370063e-10, 0.576966345)
           CFrameMon = CFrame.new(-1895.05359, 7.28907108, -2758.38428)
        elseif MyLevel == 275 or MyLevel <= 299 then -- Gladiato
           Ms = "Gladiator [Lv. 275]"
           NaemQuest = "ColosseumQuest"
           LevelQuest = 2
           NameMon = "Gladiato"
           CFrameQuest = CFrame.new(-1576.11743, 7.38933945, -2983.30762, 0.576966345, 1.22114863e-09, 0.816767931, -3.58496594e-10, 1, -1.24185606e-09, -0.816767931, 4.2370063e-10, 0.576966345)
           CFrameMon = CFrame.new(-1270.75, 7.38933659, -3321.20752)
        elseif MyLevel == 300 or MyLevel <= 329 then -- Military Soldier
           Ms = "Military Soldier [Lv. 300]"
           NaemQuest = "MagmaQuest"
           LevelQuest = 1
           NameMon = "Military Soldier"
           CFrameQuest = CFrame.new(-5316.55859, 12.2370615, 8517.2998, 0.588437557, -1.37880001e-08, -0.808542669, -2.10116209e-08, 1, -3.23446478e-08, 0.808542669, 3.60215964e-08, 0.588437557)
           CFrameMon = CFrame.new(-5411.63965, 11.0802879, 8454.12012)
        elseif MyLevel == 300 or MyLevel <= 374 then -- Military Spy
           Ms = "Military Spy [Lv. 330]"
           NaemQuest = "MagmaQuest"
           LevelQuest = 2
           NameMon = "Military Spy"
           CFrameQuest = CFrame.new(-5316.55859, 12.2370615, 8517.2998, 0.588437557, -1.37880001e-08, -0.808542669, -2.10116209e-08, 1, -3.23446478e-08, 0.808542669, 3.60215964e-08, 0.588437557)
           CFrameMon = CFrame.new(-5836.07959, 77.2306595, 8827.66211)
        elseif MyLevel == 375 or MyLevel <= 399 then -- Fishman Warrior
           Ms = "Fishman Warrior [Lv. 375]"
           NaemQuest = "FishmanQuest"
           LevelQuest = 1
           NameMon = "Fishman Warrior"
           CFrameQuest = CFrame.new(61122.5625, 18.4716396, 1568.16504, 0.893533468, 3.95251609e-09, 0.448996574, -2.34327455e-08, 1, 3.78297464e-08, -0.448996574, -4.43233645e-08, 0.893533468)
           CFrameMon = CFrame.new(60911.8906, 18.4828224, 1585.7478)
        elseif MyLevel == 400 or MyLevel <= 449 then -- Fishman Commando
           Ms = "Fishman Commando [Lv. 400]"
           NaemQuest = "FishmanQuest"
           LevelQuest = 2
           NameMon = "Fishman Commando"
           CFrameQuest = CFrame.new(61122.5625, 18.4716396, 1568.16504, 0.893533468, 3.95251609e-09, 0.448996574, -2.34327455e-08, 1, 3.78297464e-08, -0.448996574, -4.43233645e-08, 0.893533468)
           CFrameMon = CFrame.new(61916.8438, 18.4828224, 1444.38635)
        elseif MyLevel == 450 or MyLevel <= 474 then -- God's Guards
           Ms = "God's Guard [Lv. 450]"
           NaemQuest = "SkyExp1Quest"
           LevelQuest = 1
           NameMon = "God's Guards"
           CFrameQuest = CFrame.new(-4721.71436, 845.277161, -1954.20105, -0.999277651, -5.56969759e-09, 0.0380011722, -4.14751478e-09, 1, 3.75035256e-08, -0.0380011722, 3.73188307e-08, -0.999277651)
           CFrameMon = CFrame.new(-4711.30566, 845.277161, -1902.31335)
        elseif MyLevel == 475 or MyLevel <= 524 then -- Shandas
           Ms = "Shanda [Lv. 475]"
           NaemQuest = "SkyExp1Quest"
           LevelQuest = 2
           NameMon = "Shandas"
           CFrameQuest = CFrame.new(-7863.63672, 5545.49316, -379.826324, 0.362120807, -1.98046344e-08, -0.93213129, 4.05822291e-08, 1, -5.48095125e-09, 0.93213129, -3.58431969e-08, 0.362120807)
           CFrameMon = CFrame.new(-7666.93408, 5545.49316, -484.052582)
        elseif MyLevel == 525 or MyLevel <= 549 then -- Royal Squad
           Ms = "Royal Squad [Lv. 525]"
           NaemQuest = "SkyExp2Quest"
           LevelQuest = 1
           NameMon = "Royal Squad"
           CFrameQuest = CFrame.new(-7902.66895, 5635.96387, -1411.71802, 0.0504222959, 2.5710392e-08, 0.998727977, 1.12541557e-07, 1, -3.14249675e-08, -0.998727977, 1.13982921e-07, 0.0504222959)
           CFrameMon = CFrame.new(-7684.97266, 5606.87842, -1454.46094)
        elseif MyLevel == 550 or MyLevel <= 624 then -- Royal Soldier
           Ms = "Royal Soldier [Lv. 550]"
           NaemQuest = "SkyExp2Quest"
           LevelQuest = 2
           NameMon = "Royal Soldier"
           CFrameQuest = CFrame.new(-7902.66895, 5635.96387, -1411.71802, 0.0504222959, 2.5710392e-08, 0.998727977, 1.12541557e-07, 1, -3.14249675e-08, -0.998727977, 1.13982921e-07, 0.0504222959)
           CFrameMon = CFrame.new(-7836.83936, 5606.87842, -1745.65442)
        elseif MyLevel == 625 or MyLevel <= 649 then -- Galley Pirate
           Ms = "Galley Pirate [Lv. 625]"
           NaemQuest = "FountainQuest"
           LevelQuest = 1
           NameMon = "Galley Pirate"
           CFrameQuest = CFrame.new(5254.60156, 38.5011406, 4049.69678, -0.0504891425, -3.62066501e-08, -0.998724639, -9.87921389e-09, 1, -3.57534553e-08, 0.998724639, 8.06145284e-09, -0.0504891425)
           CFrameMon = CFrame.new(5584.53809, 38.5386353, 3979.62256)
        elseif MyLevel >= 650 then -- Galley Captain
           Ms = "Galley Captain [Lv. 650]"
           NaemQuest = "FountainQuest"
           LevelQuest = 2
           NameMon = "Galley Captain"
           CFrameQuest = CFrame.new(5254.60156, 38.5011406, 4049.69678, -0.0504891425, -3.62066501e-08, -0.998724639, -9.87921389e-09, 1, -3.57534553e-08, 0.998724639, 8.06145284e-09, -0.0504891425)
           CFrameMon = CFrame.new(5778.60547, 60.1919327, 4915.27734)
         end
      end
      if SecondWorld then
         if MyLevel == 700 or MyLevel <= 724 then -- Raider [Lv. 700]
            Ms = "Raider [Lv. 700]"
            NaemQuest = "Area1Quest"
            LevelQuest = 1
            NameMon = "Raider"
            CFrameQuest = CFrame.new(-424.080078, 73.0055847, 1836.91589, 0.253544956, -1.42165932e-08, 0.967323601, -6.00147771e-08, 1, 3.04272909e-08, -0.967323601, -6.5768397e-08, 0.253544956)
            CFrameMon = CFrame.new(-727.515747, 39.1748352, 2391.43286)
         elseif MyLevel == 725 or MyLevel <= 774 then -- Mercenary [Lv. 725]
            Ms = "Mercenary [Lv. 725]"
            NaemQuest = "Area1Quest"
            LevelQuest = 2
            NameMon = "Mercenary"
            CFrameQuest = CFrame.new(-424.080078, 73.0055847, 1836.91589, 0.253544956, -1.42165932e-08, 0.967323601, -6.00147771e-08, 1, 3.04272909e-08, -0.967323601, -6.5768397e-08, 0.253544956)
            CFrameMon = CFrame.new(-994.678772, 73.3529816, 1415.89807)
         elseif MyLevel == 775 or MyLevel <= 799 then -- Swan Pirate [Lv. 775]
            Ms = "Swan Pirate [Lv. 775]"
            NaemQuest = "Area2Quest"
            LevelQuest = 1
            NameMon = "Swan Pirate"
            CFrameQuest = CFrame.new(632.698608, 73.1055908, 918.666321, -0.0319722369, 8.96074881e-10, -0.999488771, 1.36326533e-10, 1, 8.92172336e-10, 0.999488771, -1.07732087e-10, -0.0319722369)
            CFrameMon = CFrame.new(934.597229, 73.3029785, 1250.96814)
         elseif MyLevel == 800 or MyLevel <= 874 then -- Factory Staff [Lv. 800] normal
            Ms = "Factory Staff [Lv. 800]"
            NaemQuest = "Area2Quest"
            LevelQuest = 2
            NameMon = "Factory Staff"
            CFrameQuest = CFrame.new(632.698608, 73.1055908, 918.666321, -0.0319722369, 8.96074881e-10, -0.999488771, 1.36326533e-10, 1, 8.92172336e-10, 0.999488771, -1.07732087e-10, -0.0319722369)
            CFrameMon = CFrame.new(676.654663, 73.3030396, 18.7464504)
         elseif MyLevel == 875 or MyLevel <= 899 then -- Marine Lieutenant [Lv. 875]
            Ms = "Marine Lieutenant [Lv. 875]"
            NaemQuest = "MarineQuest3"
            LevelQuest = 1
            NameMon = "Marine Lieutenant"
            CFrameQuest = CFrame.new(-2442.65015, 73.0511475, -3219.11523, -0.873540044, 4.2329841e-08, -0.486752301, 5.64383384e-08, 1, -1.43220786e-08, 0.486752301, -3.99823996e-08, -0.873540044)
            CFrameMon = CFrame.new(-2784.70703, 73.3094025, -2916.48779)
         elseif MyLevel == 900 or MyLevel <= 949 then -- Marine Captain [Lv. 900]
            Ms = "Marine Captain [Lv. 900]"
            NaemQuest = "MarineQuest3"
            LevelQuest = 2
            NameMon = "Marine Captain"
            CFrameQuest = CFrame.new(-2442.65015, 73.0511475, -3219.11523, -0.873540044, 4.2329841e-08, -0.486752301, 5.64383384e-08, 1, -1.43220786e-08, 0.486752301, -3.99823996e-08, -0.873540044)
            CFrameMon = CFrame.new(-1887.87061, 73.3094025, -3231.52173)
         elseif MyLevel == 950 or MyLevel <= 974 then -- Zombie [Lv. 950]
            Ms = "Zombie [Lv. 950]"
            NaemQuest = "ZombieQuest"
            LevelQuest = 1
            NameMon = "Zombie"
            CFrameQuest = CFrame.new(-5492.79395, 48.5151672, -793.710571, 0.321800292, -6.24695815e-08, 0.946807742, 4.05616092e-08, 1, 5.21931227e-08, -0.946807742, 2.16082796e-08, 0.321800292)
            CFrameMon = CFrame.new(-5674.96729, 48.5151672, -832.125366)
         elseif MyLevel == 975 or MyLevel <= 999 then -- Vampire [Lv. 975]
            Ms = "Vampire [Lv. 975]"
            NaemQuest = "ZombieQuest"
            LevelQuest = 2
            NameMon = "Vampire"
            CFrameQuest = CFrame.new(-5492.79395, 48.5151672, -793.710571, 0.321800292, -6.24695815e-08, 0.946807742, 4.05616092e-08, 1, 5.21931227e-08, -0.946807742, 2.16082796e-08, 0.321800292)
            CFrameMon = CFrame.new(-6037.83398, 6.43773842, -1295.86438)
         elseif MyLevel == 1000 or MyLevel <= 1049 then -- Snow Trooper [Lv. 1000] **
            Ms = "Snow Trooper [Lv. 1000]"
            NaemQuest = "SnowMountainQuest"
            LevelQuest = 1
            NameMon = "Snow Trooper"
            CFrameQuest = CFrame.new(604.964966, 401.457062, -5371.69287, 0.353063971, 1.89520435e-08, -0.935599446, -5.81846002e-08, 1, -1.70033754e-09, 0.935599446, 5.50377841e-08, 0.353063971)
            CFrameMon = CFrame.new(606.887939, 401.457062, -5558.42578)
         elseif MyLevel == 1050 or MyLevel <= 1099 then -- Winter Warrior [Lv. 1050]
            Ms = "Winter Warrior [Lv. 1050]"
            NaemQuest = "SnowMountainQuest"
            LevelQuest = 2
            NameMon = "Winter Warrior"
            CFrameQuest = CFrame.new(604.964966, 401.457062, -5371.69287, 0.353063971, 1.89520435e-08, -0.935599446, -5.81846002e-08, 1, -1.70033754e-09, 0.935599446, 5.50377841e-08, 0.353063971)
            CFrameMon = CFrame.new(1187.3241, 429.457001, -5198.05957)
         elseif MyLevel == 1100 or MyLevel <= 1124 then -- Lab Subordinate [Lv. 1100]
            Ms = "Lab Subordinate [Lv. 1100]"
            NaemQuest = "IceSideQuest"
            LevelQuest = 1
            NameMon = "Lab Subordinate"
            CFrameQuest = CFrame.new(-6060.10693, 15.9868021, -4904.7876, -0.411000341, -5.06538868e-07, 0.91163528, 1.26306062e-07, 1, 6.12581289e-07, -0.91163528, 3.66916197e-07, -0.411000341)
            CFrameMon = CFrame.new(-5769.36963, 15.9867992, -4448.93457)
         elseif MyLevel == 1125 or MyLevel <= 1174 then -- Horned Warrior [Lv. 1125]
            Ms = "Horned Warrior [Lv. 1125]"
            NaemQuest = "IceSideQuest"
            LevelQuest = 2
            NameMon = "Horned Warrior"
            CFrameQuest = CFrame.new(-6060.10693, 15.9868021, -4904.7876, -0.411000341, -5.06538868e-07, 0.91163528, 1.26306062e-07, 1, 6.12581289e-07, -0.91163528, 3.66916197e-07, -0.411000341)
            CFrameMon = CFrame.new(-6382.15332, 15.9867992, -5773.10303)
         elseif MyLevel == 1175 or MyLevel <= 1199 then -- Magma Ninja [Lv. 1175]
            Ms = "Magma Ninja [Lv. 1175]"
            NaemQuest = "FireSideQuest"
            LevelQuest = 1
            NameMon = "Magma Ninja"
            CFrameQuest = CFrame.new(-5431.09473, 15.9868021, -5296.53223, 0.831796765, 1.15322464e-07, -0.555080295, -1.10814341e-07, 1, 4.17010995e-08, 0.555080295, 2.68240168e-08, 0.831796765)
            CFrameMon = CFrame.new(-5475.24, 80.5986, -5970.54)
         elseif MyLevel == 1200 or MyLevel <= 1249 then -- Lava Pirate [Lv. 1200]
            Ms = "Lava Pirate [Lv. 1200]"
            NaemQuest = "FireSideQuest"
            LevelQuest = 2
            NameMon = "Lava Pirate"
            CFrameQuest = CFrame.new(-5431.09473, 15.9868021, -5296.53223, 0.831796765, 1.15322464e-07, -0.555080295, -1.10814341e-07, 1, 4.17010995e-08, 0.555080295, 2.68240168e-08, 0.831796765)
            CFrameMon = CFrame.new(-5224.03027, 53.8534698, -4752.47217)
         elseif MyLevel == 1250 or MyLevel <= 1274 then -- Ship Deckhand [Lv. 1250]
            Ms = "Ship Deckhand [Lv. 1250]"
            NaemQuest = "ShipQuest1"
            LevelQuest = 1
            NameMon = "Ship Deckhand"
            CFrameQuest = CFrame.new(1037.80127, 125.092171, 32911.6016, -0.244533166, -0, -0.969640911, -0, 1.00000012, -0, 0.96964103, 0, -0.244533136)
            CFrameMon = CFrame.new(1209.69299, 125.092163, 33051.5352)
         elseif MyLevel == 1275 or MyLevel <= 1299 then -- Ship Engineer [Lv. 1275] normal
            Ms = "Ship Engineer [Lv. 1275]"
            NaemQuest = "ShipQuest1"
            LevelQuest = 2
            NameMon = "Ship Engineer"
            CFrameQuest = CFrame.new(1037.80127, 125.092171, 32911.6016, -0.244533166, -0, -0.969640911, -0, 1.00000012, -0, 0.96964103, 0, -0.244533136)
            CFrameMon = CFrame.new(950.67041, 40.4779129, 32820.293)
         elseif MyLevel == 1300 or MyLevel <= 1324 then -- Ship Steward [Lv. 1300]
            Ms = "Ship Steward [Lv. 1300]"
            NaemQuest = "ShipQuest2"
            LevelQuest = 1
            NameMon = "Ship Steward"
            CFrameQuest = CFrame.new(968.80957, 125.092171, 33244.125, -0.869560242, 1.51905191e-08, -0.493826836, 1.44108379e-08, 1, 5.38534195e-09, 0.493826836, -2.43357912e-09, -0.869560242)
            CFrameMon = CFrame.new(918.382141, 129.899292, 33441.6484)
         elseif MyLevel == 1325 or MyLevel <= 1349 then -- Ship Officer [Lv. 1325] normal
            Ms = "Ship Officer [Lv. 1325]"
            NaemQuest = "ShipQuest2"
            LevelQuest = 2
            NameMon = "Ship Officer"
            CFrameQuest = CFrame.new(968.80957, 125.092171, 33244.125, -0.869560242, 1.51905191e-08, -0.493826836, 1.44108379e-08, 1, 5.38534195e-09, 0.493826836, -2.43357912e-09, -0.869560242)
            CFrameMon = CFrame.new(1188.86938, 181.184189, 33303)
         elseif MyLevel == 1350 or MyLevel <= 1374 then -- Arctic Warrior [Lv. 1350]
            Ms = "Arctic Warrior [Lv. 1350]"
            NaemQuest = "FrostQuest"
            LevelQuest = 1
            NameMon = "Arctic Warrior"
            CFrameQuest = CFrame.new(5669.43506, 28.2117786, -6482.60107, 0.888092756, 1.02705066e-07, 0.459664226, -6.20391774e-08, 1, -1.03572376e-07, -0.459664226, 6.34646895e-08, 0.888092756)
            CFrameMon = CFrame.new(6044.61426, 28.4024639, -6214.63232)
         elseif MyLevel == 1375 or MyLevel <= 1424 then -- Snow Lurker [Lv. 1375]
            Ms = "Snow Lurker [Lv. 1375]"
            NaemQuest = "FrostQuest"
            LevelQuest = 2
            NameMon = "Snow Lurker"
            CFrameQuest = CFrame.new(5669.43506, 28.2117786, -6482.60107, 0.888092756, 1.02705066e-07, 0.459664226, -6.20391774e-08, 1, -1.03572376e-07, -0.459664226, 6.34646895e-08, 0.888092756)
            CFrameMon = CFrame.new(5645.45, 46.6215, -6512.57)
          elseif MyLevel == 1425 or MyLevel <= 1449 then -- Sea Soldier [Lv. 1425]
             Ms = "Sea Soldier [Lv. 1425]"
             NaemQuest = "ForgottenQuest"
             LevelQuest = 1
             NameMon = "Sea Soldier"
             CFrameQuest = CFrame.new(-3052.99097, 236.881363, -10148.1943, -0.997911751, 4.42321983e-08, 0.064591676, 4.90968759e-08, 1, 7.37270085e-08, -0.064591676, 7.67442998e-08, -0.997911751)
             CFrameMon = CFrame.new(-3375.28076, 27.2922382, -9779.68262)
          elseif MyLevel >= 1450 then -- Water Fighter [Lv. 1450]
             Ms = "Water Fighter [Lv. 1450]"
             NaemQuest = "ForgottenQuest"
             LevelQuest = 2
             NameMon = "Water Fighter"
             CFrameQuest = CFrame.new(-3052.99097, 236.881363, -10148.1943, -0.997911751, 4.42321983e-08, 0.064591676, 4.90968759e-08, 1, 7.37270085e-08, -0.064591676, 7.67442998e-08, -0.997911751)
             CFrameMon = CFrame.new(-3471.76123, 239.189392, -10523.5547)
         end
      end
      if ThirdWorld then
        if MyLevel == 1500 or MyLevel <= 1524 then -- Raider [Lv. 700]
           Ms = "Pirate Millionaire [Lv. 1500]"
           NaemQuest = "PiratePortQuest"
           LevelQuest = 1
           NameMon = "Pirate Millionaire"
           CFrameQuest = CFrame.new(-288.95349121094, 43.789222717285, 5578.7622070313)
           CFrameMon = CFrame.new(-560.434387, 43.8105583, 5532.06885, 0.987512887, -1.65048668e-08, 0.157538295, 2.30374742e-08, 1, -3.96407493e-08, -0.157538295, 4.27750351e-08, 0.987512887)

        elseif MyLevel == 1525 or MyLevel <= 1574 then
            Ms = "Pistol Billionaire [Lv. 1525]"
            NaemQuest = "PiratePortQuest"
            LevelQuest = 2
            NameMon = "Pistol Billionaire"
            CFrameQuest = CFrame.new(-288.95349121094, 43.789222717285, 5578.7622070313)
            CFrameMon = CFrame.new(-52.5135803, 73.8293839, 5988.42676, 0.543421149, 0, -0.839460254, -0, 1, -0, 0.839460254, 0, 0.543421149)

        elseif MyLevel == 1575 or MyLevel <= 1599 then
            Ms = "Dragon Crew Warrior [Lv. 1575]"
            NaemQuest = "AmazonQuest"
            LevelQuest = 1
            NameMon = "Dragon Crew Warrior"
            CFrameQuest = CFrame.new(5833.5776367188, 51.575191497803, -1102.7550048828)
            CFrameMon = CFrame.new(2483.66724, 73.1250763, -7454.39258, 0.999753118, 0, -0.0222221594, -0, 1, -0, 0.0222221613, 0, 0.999752998)

        elseif MyLevel == 1600 or MyLevel <= 1624 then
            Ms = "Dragon Crew Archer [Lv. 1600]"
            NaemQuest = "AmazonQuest"
            LevelQuest = 2
            NameMon = "Dragon Crew Archer"
            CFrameQuest = CFrame.new(5833.5776367188, 51.575191497803, -1102.7550048828)
            CFrameMon = CFrame.new(6348.09424, 51.4962807, -1175.75098, 0.334417313, 0, 0.942425072, -0, 1, -0, -0.942425191, 0, 0.334417284)

        elseif MyLevel == 1625 or MyLevel <= 1649 then
            Ms = "Female Islander [Lv. 1625]"
            NaemQuest = "AmazonQuest2"
            LevelQuest = 1
            NameMon = "Female Islander"
            CFrameQuest = CFrame.new(5447.388671875, 601.74407958984, 751.86462402344)
            CFrameMon = CFrame.new(4743.52881, 744.384216, 741.278687, 0.731631517, 0, -0.681700349, -0, 1, -0, 0.681700408, 0, 0.731631458)

        elseif MyLevel == 1650 or MyLevel <= 1699 then
            Ms = "Giant Islander [Lv. 1650]"
            NaemQuest = "AmazonQuest2"
            LevelQuest = 2
            NameMon = "Giant Islander"
            CFrameQuest = CFrame.new(5447.388671875, 601.74407958984, 751.86462402344)
            CFrameMon = CFrame.new(4815.85693, 676.570313, -95.0762863, 0.981796861, 0, 0.189934134, -0, 1, -0, -0.189934134, 0, 0.981796861)

        elseif MyLevel == 1700 or MyLevel <= 1724 then
            Ms = "Marine Commodore [Lv. 1700]"
            NaemQuest = "MarineTreeIsland"
            LevelQuest = 1
            NameMon = "Marine Commodore"
            CFrameQuest = CFrame.new(2179.2548828125, 28.701448440552, -6739.7299804688)
            CFrameMon = CFrame.new(2483.66724, 73.1250763, -7454.39258, 0.999753118, 0, -0.0222221594, -0, 1, -0, 0.0222221613, 0, 0.999752998)

        elseif MyLevel == 1725 or MyLevel <= 1774 then
            Ms = "Marine Rear Admiral [Lv. 1725]"
            NaemQuest = "MarineTreeIsland"
            LevelQuest = 2
            NameMon = "Marine Rear Admiral"
            CFrameQuest = CFrame.new(2179.2548828125, 28.701448440552, -6739.7299804688)
            CFrameMon = CFrame.new(3603.6355, 160.524094, -6984.63574, 0.999736547, 0, -0.0229543783, -0, 0.99999994, -0, 0.0229543801, 0, 0.999736428)
            
        elseif MyLevel == 1775 or MyLevel <= 1799 then
            Ms = "Fishman Raider [Lv. 1775]"
            NaemQuest = "DeepForestIsland3"
            LevelQuest = 1
            NameMon = "Fishman Raider"
            CFrameQuest = CFrame.new(-10580.998046875, 331.75863647461, -8758.193359375)
            CFrameMon = CFrame.new(-10574.4756, 331.762726, -8477.02148, -0.424698561, 0, 0.90533489, 0, 1, -0, -0.905334771, 0, -0.424698621)
        elseif MyLevel == 1800 or MyLevel <= 1824 then
            Ms = "Fishman Captain [Lv. 1800]"
            NaemQuest = "DeepForestIsland3"
            LevelQuest = 2
            NameMon = "Fishman Captain"
            CFrameQuest = CFrame.new(-10580.998046875, 331.75863647461, -8758.193359375)
            CFrameMon = CFrame.new(-11003.832, 331.762726, -8888.6748, -0.996632755, -0, -0.0819952488, -0, 1, -0, 0.0819952488, 0, -0.996632755)

        elseif MyLevel == 1825 or MyLevel <= 1849 then
            Ms = "Forest Pirate [Lv. 1825]"
            NaemQuest = "DeepForestIsland"
            LevelQuest = 1
            NameMon = "Forest Pirate"
            CFrameQuest = CFrame.new(-13231.467773438, 332.37414550781, -7626.6860351563)
            CFrameMon = CFrame.new(-13414.0039, 332.378235, -7844.8501, 0.592847407, 0, -0.805314839, -0, 1, -0, 0.805314839, 0, 0.592847407)

        elseif MyLevel == 1850 or MyLevel <= 1899 then
            Ms = "Mythological Pirate [Lv. 1850]"
            NaemQuest = "DeepForestIsland"
            LevelQuest = 2
            NameMon = "Mythological Pirate"
            CFrameQuest = CFrame.new(-13231.467773438, 332.37414550781, -7626.6860351563)
            CFrameMon = CFrame.new(-13591.3984, 469.788422, -6977.63379, -0.970228612, -0, -0.242190942, -0, 1, -0, 0.242190942, 0, -0.970228612)
        elseif MyLevel == 1900 or MyLevel <= 1924 then
            Ms = "Jungle Pirate [Lv. 1900]"
            NaemQuest = "DeepForestIsland2"
            LevelQuest = 1
            NameMon = "Jungle Pirate"
            CFrameQuest = CFrame.new(-12683.189453125, 390.85668945313, -9902.15625)
            CFrameMon = CFrame.new(-11975.2871, 331.738373, -10653.6221, -0.324082136, 1.34216107e-05, -0.946028888, 2.08913252e-06, 1, 1.34716402e-05, 0.946028888, 2.38953862e-06, -0.324082136)

        elseif MyLevel == 1925 or MyLevel <= 1974 then
            Ms = "Musketeer Pirate [Lv. 1925]"
            NaemQuest = "DeepForestIsland2"
            LevelQuest = 2
            NameMon = "Musketeer Pirate"
            CFrameQuest = CFrame.new(-12683.189453125, 390.85668945313, -9902.15625)
            CFrameMon = CFrame.new(-13367.6836, 391.545746, -9808.33008, 0.327346772, -0.000111054818, 0.94490397, 3.52763564e-05, 1, 0.000105309307, -0.94490397, -1.13990154e-06, 0.327346772)
         elseif MyLevel == 1975 or MyLevel <= 1999 then
            Ms = "Reborn Skeleton [Lv. 1975]"
            NaemQuest = "HauntedQuest1"
            LevelQuest = 1
            NameMom = "Reborn Skeleton"
            CFrameQuest = CFrame.new(-9490.2793, 142.104858, 5565.8501)
            CFrameMon = CFrame.new(-8773.98828, 142.104813, 5967.98535, -0.992157221, -0, -0.124996185, -0, 1, -0, 0.124996185, 0, -0.992157221)

         elseif MyLevel == 2000 or MyLevel <= 2024 then
            Ms = "Living Zombie [Lv. 2000]"
            NaemQuest = "HauntedQuest1"
            LevelQuest = 2
            NameMon = "Living Zombie"
            CFrameQuest = CFrame.new(-9490.2793, 142.104858, 5565.8501)
            CFrameMon = CFrame.new(-10097.0674, 146.005447, 5911.93701, 0.146000057, 0, 0.989284575, -0, 1.00000012, -0, -0.989284694, 0, 0.146000043)

         elseif MyLevel == 2025 or MyLevel <= 2049 then
            Ms = "Demonic Soul [Lv. 2025]"
            NaemQuest = "HauntedQuest2"
            LevelQuest = 1
            NameMon = "Demonic Soul"
            CFrameQuest = CFrame.new(-9506.95313, 172.104858, 6074.63086)
            CFrameMon = CFrame.new(-9502.33008, 172.104858, 6159.75488, 0.99965167, -7.1589441e-09, -0.0263923369, 5.66633451e-09, 1, -5.6629446e-08, 0.0263923369, 5.64601734e-08, 0.99965167)

         elseif MyLevel >= 2050 then
            Ms = "Posessed Mummy [Lv. 2050]" 
            NaemQuest = "HauntedQuest2"
            LevelQuest = 2
            NameMon = "Posessed Mummy"
            CFrameQuest = CFrame.new(-9506.95313, 172.104858, 6074.63086)
            CFrameMon = CFrame.new(-9567.90625, 5.79253197, 6218.73389, -0.999981761, 4.25995026e-08, -0.00604327489, 4.22182644e-08, 1, 6.3212056e-08, 0.00604327489, 6.29557633e-08, -0.999981761)
        end
     end
 end

MobsPick = ""

function CheckQuest2()
   if ThirdWorld then
      if MobsPick == "Pirate Millionaire [Lv. 1500]" then
         NaemQuestm = "PiratePortQuest"
         LevelQuestm = 1
         NameMonm = "Pirate Millionaire"
         CFrameQuestm = CFrame.new(-288.95349121094, 43.789222717285, 5578.7622070313)
         CFrameMonm = CFrame.new(-560.434387, 43.8105583, 5532.06885, 0.987512887, -1.65048668e-08, 0.157538295, 2.30374742e-08, 1, -3.96407493e-08, -0.157538295, 4.27750351e-08, 0.987512887)

      elseif MobsPick == "Pistol Billionaire [Lv. 1525]" then
          NaemQuestm = "PiratePortQuest"
          LevelQuestm = 2
          NameMonm = "Pistol Billionaire"
          CFrameQuestm = CFrame.new(-288.95349121094, 43.789222717285, 5578.7622070313)
          CFrameMonm = CFrame.new(2483.66724, 73.1250763, -7454.39258, 0.999753118, 0, -0.0222221594, -0, 1, -0, 0.0222221613, 0, 0.999752998)

      elseif MobsPick == "Dragon Crew Warrior [Lv. 1575]" then
          NaemQuestm = "AmazonQuest"
          LevelQuestm = 1
          NameMonm = "Dragon Crew Warrior"
          CFrameQuestm = CFrame.new(5833.5776367188, 51.575191497803, -1102.7550048828)
          CFrameMonm = CFrame.new(6348.09424, 51.4962807, -1175.75098, 0.334417313, 0, 0.942425072, -0, 1, -0, -0.942425191, 0, 0.334417284)

      elseif MobsPick == "Dragon Crew Archer [Lv. 1600]" then
          NaemQuestm = "AmazonQuest"
          LevelQuestm = 2
          NameMonm = "Dragon Crew Archer"
          CFrameQuestm = CFrame.new(5833.5776367188, 51.575191497803, -1102.7550048828)
          CFrameMonm = CFrame.new(6348.09424, 51.4962807, -1175.75098, 0.334417313, 0, 0.942425072, -0, 1, -0, -0.942425191, 0, 0.334417284)

      elseif MobsPick == "Female Islander [Lv. 1625]"  then
          NaemQuestm = "AmazonQuest2"
          LevelQuestm = 1
          NameMonm = "Female Islander"
          CFrameQuestm = CFrame.new(5447.388671875, 601.74407958984, 751.86462402344)
          CFrameMonm = CFrame.new(4743.52881, 744.384216, 741.278687, 0.731631517, 0, -0.681700349, -0, 1, -0, 0.681700408, 0, 0.731631458)

      elseif MobsPick == "Giant Islander [Lv. 1650]" then
          NaemQuestm = "AmazonQuest2"
          LevelQuestm = 2
          NameMonm = "Giant Islander"
          CFrameQuestm = CFrame.new(5447.388671875, 601.74407958984, 751.86462402344)
          CFrameMonm = CFrame.new(4815.85693, 676.570313, -95.0762863, 0.981796861, 0, 0.189934134, -0, 1, -0, -0.189934134, 0, 0.981796861)

      elseif MobsPick == "Marine Commodore [Lv. 1700]"  then
          NaemQuestm = "MarineTreeIsland"
          LevelQuestm = 1
          NameMonm = "Marine Commodore"
          CFrameQuestm = CFrame.new(2179.2548828125, 28.701448440552, -6739.7299804688)
          CFrameMonm = CFrame.new(2483.66724, 73.1250763, -7454.39258, 0.999753118, 0, -0.0222221594, -0, 1, -0, 0.0222221613, 0, 0.999752998)

      elseif MobsPick == "Marine Rear Admiral [Lv. 1725]"  then
          NaemQuestm = "MarineTreeIsland"
          LevelQuestm = 2
          NameMonm = "Marine Rear Admiral"
          CFrameQuestm = CFrame.new(2179.2548828125, 28.701448440552, -6739.7299804688)
          CFrameMonm = CFrame.new(3603.6355, 160.524094, -6984.63574, 0.999736547, 0, -0.0229543783, -0, 0.99999994, -0, 0.0229543801, 0, 0.999736428)

      elseif MobsPick == "Fishman Raider [Lv. 1775]"  then
          NaemQuestm = "DeepForestIsland3"
          LevelQuestm = 1
          NameMonm = "Fishman Raider"
          CFrameQuestm = CFrame.new(-10580.998046875, 331.75863647461, -8758.193359375)
          CFrameMonm = CFrame.new(-10574.4756, 331.762726, -8477.02148, -0.424698561, 0, 0.90533489, 0, 1, -0, -0.905334771, 0, -0.424698621)

      elseif MobsPick == "Fishman Captain [Lv. 1800]"  then
          NaemQuestm = "DeepForestIsland3"
          LevelQuestm = 2
          NameMonm = "Fishman Captain"
          CFrameQuestm = CFrame.new(-10580.998046875, 331.75863647461, -8758.193359375)
          CFrameMonm = CFrame.new(-11003.832, 331.762726, -8888.6748, -0.996632755, -0, -0.0819952488, -0, 1, -0, 0.0819952488, 0, -0.996632755)

      elseif MobsPick == "Forest Pirate [Lv. 1825]"  then
          NaemQuestm = "DeepForestIsland"
          LevelQuestm = 1
          NameMonm = "Forest Pirate"
          CFrameQuestm = CFrame.new(-13231.467773438, 332.37414550781, -7626.6860351563)
          CFrameMonm = CFrame.new(-13414.0039, 332.378235, -7844.8501, 0.592847407, 0, -0.805314839, -0, 1, -0, 0.805314839, 0, 0.592847407)

      elseif MobsPick == "Mythological Pirate [Lv. 1850]"  then
          NaemQuestm = "DeepForestIsland"
          LevelQuestm = 2
          NameMonm = "Mythological Pirate"
          CFrameQuestm = CFrame.new(-13231.467773438, 332.37414550781, -7626.6860351563)
          CFrameMonm = CFrame.new(-13591.3984, 469.788422, -6977.63379, -0.970228612, -0, -0.242190942, -0, 1, -0, 0.242190942, 0, -0.970228612)

      elseif MobsPick == "Jungle Pirate [Lv. 1900]"  then
          NaemQuestm = "DeepForestIsland2"
          LevelQuestm = 1
          NameMonm = "Jungle Pirate"
          CFrameQuestm = CFrame.new(-12683.189453125, 390.85668945313, -9902.15625)
          CFrameMonm = CFrame.new(-11975.2871, 331.738373, -10653.6221, -0.324082136, 1.34216107e-05, -0.946028888, 2.08913252e-06, 1, 1.34716402e-05, 0.946028888, 2.38953862e-06, -0.324082136)

      elseif MobsPick == "Musketeer Pirate [Lv. 1925]"  then
          NaemQuestm = "DeepForestIsland2"
          LevelQuestm = 2
          NameMonm = "Musketeer Pirate"
          CFrameQuestm = CFrame.new(-12683.189453125, 390.85668945313, -9902.15625)
          CFrameMonm = CFrame.new(-13367.6836, 391.545746, -9808.33008, 0.327346772, -0.000111054818, 0.94490397, 3.52763564e-05, 1, 0.000105309307, -0.94490397, -1.13990154e-06, 0.327346772)

      elseif MobsPick == "Reborn Skeleton [Lv. 1975]" then
         NaemQuestm = "HauntedQuest1"
         LevelQuestm = 1
         NameMonm = "Reborn Skeleton"
         CFrameQuestm = CFrame.new(-9490.2793, 142.104858, 5565.8501)
         CFrameMonm = CFrame.new(-8773.98828, 142.104813, 5967.98535, -0.992157221, -0, -0.124996185, -0, 1, -0, 0.124996185, 0, -0.992157221)

      elseif MobsPick == "Living Zombie [Lv. 2000]" then
         NaemQuestm = "HauntedQuest1"
         LevelQuestm = 2
         NameMonm = "Living Zombie"
         CFrameQuestm = CFrame.new(-9490.2793, 142.104858, 5565.8501)
         CFrameMonm = CFrame.new(-10097.0674, 146.005447, 5911.93701, 0.146000057, 0, 0.989284575, -0, 1.00000012, -0, -0.989284694, 0, 0.146000043)
      elseif MobsPick == "Demonic Soul [Lv. 2025]" then
         NaemQuestm = "HauntedQuest2"
         LevelQuestm = 1
         NameMonm = "Demonic Soul"
         CFrameQuestm = CFrame.new(-9506.95313, 172.104858, 6074.63086)
         CFrameMonm = CFrame.new(-9502.33008, 172.104858, 6159.75488, 0.99965167, -7.1589441e-09, -0.0263923369, 5.66633451e-09, 1, -5.6629446e-08, 0.0263923369, 5.64601734e-08, 0.99965167)
      elseif MobsPick == "Posessed Mummy [Lv. 2050]" then
         NaemQuestm = "HauntedQuest2"
         LevelQuestm = 2
         NameMonm = "Posessed Mummy"
         CFrameQuestm = CFrame.new(-9506.95313, 172.104858, 6074.63086)
         CFrameMonm = CFrame.new(-9567.90625, 5.79253197, 6218.73389, -0.999981761, 4.25995026e-08, -0.00604327489, 4.22182644e-08, 1, 6.3212056e-08, 0.00604327489, 6.29557633e-08, -0.999981761)
      end
   elseif SecondWorld then
      if MobsPick == "Raider [Lv. 700]" then -- Raider [Lv. 700]
         NaemQuestm = "Area1Quest"
         LevelQuestm= 1
         NameMonm = "Raider"
         CFrameQuestm = CFrame.new(-424.080078, 73.0055847, 1836.91589, 0.253544956, -1.42165932e-08, 0.967323601, -6.00147771e-08, 1, 3.04272909e-08, -0.967323601, -6.5768397e-08, 0.253544956)
         CFrameMonm = CFrame.new(-727.515747, 39.1748352, 2391.43286)
      elseif MobsPick == "Mercenary [Lv. 725]" then -- Mercenary [Lv. 725]
         NaemQuestm = "Area1Quest"
         LevelQuestm= 2
         NameMonm = "Mercenary"
         CFrameQuestm = CFrame.new(-424.080078, 73.0055847, 1836.91589, 0.253544956, -1.42165932e-08, 0.967323601, -6.00147771e-08, 1, 3.04272909e-08, -0.967323601, -6.5768397e-08, 0.253544956)
         CFrameMonm = CFrame.new(-994.678772, 73.3529816, 1415.89807)
      elseif MobsPick == "Swan Pirate [Lv. 775]" then -- Swan Pirate [Lv. 775]
         NaemQuestm = "Area2Quest"
         LevelQuestm= 1
         NameMonm = "Swan Pirate"
         CFrameQuestm = CFrame.new(632.698608, 73.1055908, 918.666321, -0.0319722369, 8.96074881e-10, -0.999488771, 1.36326533e-10, 1, 8.92172336e-10, 0.999488771, -1.07732087e-10, -0.0319722369)
         CFrameMonm = CFrame.new(934.597229, 73.3029785, 1250.96814)
      elseif MobsPick == "Factory Staff [Lv. 800]" then -- Factory Staff [Lv. 800] normal
         NaemQuestm = "Area2Quest"
         LevelQuestm= 2
         NameMonm = "Factory Staff"
         CFrameQuestm = CFrame.new(632.698608, 73.1055908, 918.666321, -0.0319722369, 8.96074881e-10, -0.999488771, 1.36326533e-10, 1, 8.92172336e-10, 0.999488771, -1.07732087e-10, -0.0319722369)
         CFrameMonm = CFrame.new(676.654663, 73.3030396, 18.7464504)
      elseif MobsPick == "Marine Lieutenant [Lv. 875]" then -- Marine Lieutenant [Lv. 875]
         NaemQuestm = "MarineQuest3"
         LevelQuestm= 1
         NameMonm = "Marine Lieutenant"
         CFrameQuestm = CFrame.new(-2442.65015, 73.0511475, -3219.11523, -0.873540044, 4.2329841e-08, -0.486752301, 5.64383384e-08, 1, -1.43220786e-08, 0.486752301, -3.99823996e-08, -0.873540044)
         CFrameMonm = CFrame.new(-2784.70703, 73.3094025, -2916.48779)
      elseif MobsPick == "Marine Captain [Lv. 900]" then -- Marine Captain [Lv. 900]
         NaemQuestm = "MarineQuest3"
         LevelQuestm= 2
         NameMonm = "Marine Captain"
         CFrameQuestm = CFrame.new(-2442.65015, 73.0511475, -3219.11523, -0.873540044, 4.2329841e-08, -0.486752301, 5.64383384e-08, 1, -1.43220786e-08, 0.486752301, -3.99823996e-08, -0.873540044)
         CFrameMonm = CFrame.new(-1887.87061, 73.3094025, -3231.52173)
      elseif MobsPick == "Zombie [Lv. 950]" then -- Zombie [Lv. 950]
         NaemQuestm = "ZombieQuest"
         LevelQuestm= 1
         NameMonm = "Zombie"
         CFrameQuestm = CFrame.new(-5492.79395, 48.5151672, -793.710571, 0.321800292, -6.24695815e-08, 0.946807742, 4.05616092e-08, 1, 5.21931227e-08, -0.946807742, 2.16082796e-08, 0.321800292)
         CFrameMonm = CFrame.new(-5674.96729, 48.5151672, -832.125366)
      elseif MobsPick == "Vampire [Lv. 975]"  then -- Vampire [Lv. 975]
         NaemQuestm = "ZombieQuest"
         LevelQuestm= 2
         NameMonm = "Vampire"
         CFrameQuestm = CFrame.new(-5492.79395, 48.5151672, -793.710571, 0.321800292, -6.24695815e-08, 0.946807742, 4.05616092e-08, 1, 5.21931227e-08, -0.946807742, 2.16082796e-08, 0.321800292)
         CFrameMonm = CFrame.new(-6037.83398, 6.43773842, -1295.86438)
      elseif MobsPick == "Snow Trooper [Lv. 1000]"  then -- Snow Trooper [Lv. 1000] **
         NaemQuestm = "SnowMountainQuest"
         LevelQuestm= 1
         NameMonm = "Snow Trooper"
         CFrameQuestm = CFrame.new(604.964966, 401.457062, -5371.69287, 0.353063971, 1.89520435e-08, -0.935599446, -5.81846002e-08, 1, -1.70033754e-09, 0.935599446, 5.50377841e-08, 0.353063971)
         CFrameMonm = CFrame.new(606.887939, 401.457062, -5558.42578)
      elseif MobsPick ==  "Winter Warrior [Lv. 1050]"  then -- Winter Warrior [Lv. 1050]
         NaemQuestm = "SnowMountainQuest"
         LevelQuestm= 2
         NameMonm = "Winter Warrior"
         CFrameQuestm = CFrame.new(604.964966, 401.457062, -5371.69287, 0.353063971, 1.89520435e-08, -0.935599446, -5.81846002e-08, 1, -1.70033754e-09, 0.935599446, 5.50377841e-08, 0.353063971)
         CFrameMonm = CFrame.new(1187.3241, 429.457001, -5198.05957)
      elseif  MobsPick == "Lab Subordinate [Lv. 1100]" then -- Lab Subordinate [Lv. 1100]
         NaemQuestm = "IceSideQuest"
         LevelQuestm= 1
         NameMonm = "Lab Subordinate"
         CFrameQuestm = CFrame.new(-6060.10693, 15.9868021, -4904.7876, -0.411000341, -5.06538868e-07, 0.91163528, 1.26306062e-07, 1, 6.12581289e-07, -0.91163528, 3.66916197e-07, -0.411000341)
         CFrameMonm = CFrame.new(-5769.36963, 15.9867992, -4448.93457)
      elseif  MobsPick ==  "Horned Warrior [Lv. 1125]" then -- Horned Warrior [Lv. 1125]
         NaemQuestm = "IceSideQuest"
         LevelQuestm= 2
         NameMonm = "Horned Warrior"
         CFrameQuestm = CFrame.new(-6060.10693, 15.9868021, -4904.7876, -0.411000341, -5.06538868e-07, 0.91163528, 1.26306062e-07, 1, 6.12581289e-07, -0.91163528, 3.66916197e-07, -0.411000341)
         CFrameMonm = CFrame.new(-6382.15332, 15.9867992, -5773.10303)
      elseif   MobsPick == "Magma Ninja [Lv. 1175]" then -- Magma Ninja [Lv. 1175]
         NaemQuestm = "FireSideQuest"
         LevelQuestm= 1
         NameMonm = "Magma Ninja"
         CFrameQuestm = CFrame.new(-5431.09473, 15.9868021, -5296.53223, 0.831796765, 1.15322464e-07, -0.555080295, -1.10814341e-07, 1, 4.17010995e-08, 0.555080295, 2.68240168e-08, 0.831796765)
         CFrameMonm = CFrame.new(-5475.24, 80.5986, -5970.54)
      elseif MobsPick == "Lava Pirate [Lv. 1200]"  then -- Lava Pirate [Lv. 1200]
         NaemQuestm = "FireSideQuest"
         LevelQuestm= 2
         NameMonm = "Lava Pirate"
         CFrameQuestm = CFrame.new(-5431.09473, 15.9868021, -5296.53223, 0.831796765, 1.15322464e-07, -0.555080295, -1.10814341e-07, 1, 4.17010995e-08, 0.555080295, 2.68240168e-08, 0.831796765)
         CFrameMonm = CFrame.new(-5224.03027, 53.8534698, -4752.47217)
      elseif MobsPick == "Ship Deckhand [Lv. 1250]" then -- Ship Deckhand [Lv. 1250]
         NaemQuestm = "ShipQuest1"
         LevelQuestm= 1
         NameMonm = "Ship Deckhand"
         CFrameQuestm = CFrame.new(1037.80127, 125.092171, 32911.6016, -0.244533166, -0, -0.969640911, -0, 1.00000012, -0, 0.96964103, 0, -0.244533136)
         CFrameMonm = CFrame.new(1209.69299, 125.092163, 33051.5352)
      elseif MobsPick ==  "Ship Engineer [Lv. 1275]" then -- Ship Engineer [Lv. 1275] normal
         NaemQuestm = "ShipQuest1"
         LevelQuestm= 2
         NameMonm = "Ship Engineer"
         CFrameQuestm = CFrame.new(1037.80127, 125.092171, 32911.6016, -0.244533166, -0, -0.969640911, -0, 1.00000012, -0, 0.96964103, 0, -0.244533136)
         CFrameMonm = CFrame.new(950.67041, 40.4779129, 32820.293)
      elseif MobsPick == "Ship Steward [Lv. 1300]" then -- Ship Steward [Lv. 1300]
         NaemQuestm = "ShipQuest2"
         LevelQuestm= 1
         NameMonm = "Ship Steward"
         CFrameQuestm = CFrame.new(968.80957, 125.092171, 33244.125, -0.869560242, 1.51905191e-08, -0.493826836, 1.44108379e-08, 1, 5.38534195e-09, 0.493826836, -2.43357912e-09, -0.869560242)
         CFrameMonm = CFrame.new(918.382141, 129.899292, 33441.6484)
      elseif MobsPick ==  "Ship Officer [Lv. 1325]" then -- Ship Officer [Lv. 1325] normal
         NaemQuestm = "ShipQuest2"
         LevelQuestm= 2
         NameMonm = "Ship Officer"
         CFrameQuestm = CFrame.new(968.80957, 125.092171, 33244.125, -0.869560242, 1.51905191e-08, -0.493826836, 1.44108379e-08, 1, 5.38534195e-09, 0.493826836, -2.43357912e-09, -0.869560242)
         CFrameMonm = CFrame.new(1188.86938, 181.184189, 33303)
      elseif MobsPick ==  "Arctic Warrior [Lv. 1350]" then -- Arctic Warrior [Lv. 1350]
         NaemQuestm = "FrostQuest"
         LevelQuestm= 1
         NameMonm = "Arctic Warrior"
         CFrameQuestm = CFrame.new(5669.43506, 28.2117786, -6482.60107, 0.888092756, 1.02705066e-07, 0.459664226, -6.20391774e-08, 1, -1.03572376e-07, -0.459664226, 6.34646895e-08, 0.888092756)
         CFrameMonm = CFrame.new(6044.61426, 28.4024639, -6214.63232)
      elseif MobsPick == "Snow Lurker [Lv. 1375]" then -- Snow Lurker [Lv. 1375]
         NaemQuestm = "FrostQuest"
         LevelQuestm= 2
         NameMonm = "Snow Lurker"
         CFrameQuestm = CFrame.new(5669.43506, 28.2117786, -6482.60107, 0.888092756, 1.02705066e-07, 0.459664226, -6.20391774e-08, 1, -1.03572376e-07, -0.459664226, 6.34646895e-08, 0.888092756)
         CFrameMonm = CFrame.new(5645.45, 46.6215, -6512.57)
      elseif MobsPick == "Sea Soldier [Lv. 1425]" then -- Sea Soldier [Lv. 1425]
         NaemQuestm = "ForgottenQuest"
         LevelQuestm= 1
         NameMonm = "Sea Soldier"
         CFrameQuestm = CFrame.new(-3052.99097, 236.881363, -10148.1943, -0.997911751, 4.42321983e-08, 0.064591676, 4.90968759e-08, 1, 7.37270085e-08, -0.064591676, 7.67442998e-08, -0.997911751)
         CFrameMonm = CFrame.new(-3375.28076, 27.2922382, -9779.68262)
      elseif MobsPick == "Water Fighter [Lv. 1450]" then -- Water Fighter [Lv. 1450]
         NaemQuestm = "ForgottenQuest"
         LevelQuestm= 2
         NameMonm = "Water Fighter"
         CFrameQuestm = CFrame.new(-3052.99097, 236.881363, -10148.1943, -0.997911751, 4.42321983e-08, 0.064591676, 4.90968759e-08, 1, 7.37270085e-08, -0.064591676, 7.67442998e-08, -0.997911751)
         CFrameMonm = CFrame.new(-3471.76123, 239.189392, -10523.5547)
      end
   elseif FirstWorld then
      if MobsPick == "Bandit [Lv. 5]" then -- Bandit
         NaemQuestm = "BanditQuest1"
         LevelQuestm= 1
         NameMonm = "Bandit"
         CFrameQuestm = CFrame.new(1061.66699, 16.5166187, 1544.52905, -0.942978859, -3.33851502e-09, 0.332852632, 7.04340497e-09, 1, 2.99841325e-08, -0.332852632, 3.06188177e-08, -0.942978859)
         CFrameMonm = CFrame.new(1124.98352, 16.3941536, 1545.57556)
      elseif MobsPick == "Monkey [Lv. 14]" then -- Monkey
         NaemQuestm = "JungleQuest"
         LevelQuestm= 1
         NameMonm = "Monkey"
         CFrameQuestm = CFrame.new(-1604.12012, 36.8521118, 154.23732, 0.0648873374, -4.70858913e-06, -0.997892559, 1.41431883e-07, 1, -4.70933674e-06, 0.997892559, 1.64442184e-07, 0.0648873374)
         CFrameMonm = CFrame.new(-1609.50916, 36.8520927, 146.766647)
      elseif MobsPick == "Gorilla [Lv. 20]" then -- Gorilla
         NaemQuestm = "JungleQuest"
         LevelQuestm= 2
         NameMonm = "Gorilla"
         CFrameQuestm = CFrame.new(-1604.12012, 36.8521118, 154.23732, 0.0648873374, -4.70858913e-06, -0.997892559, 1.41431883e-07, 1, -4.70933674e-06, 0.997892559, 1.64442184e-07, 0.0648873374)
         CFrameMonm = CFrame.new(-1198.01721, 7.7187376, -448.22168)
      elseif MobsPick == "Pirate [Lv. 35]" then -- Pirate
         NaemQuestm = "BuggyQuest1"
         LevelQuestm= 1
         NameMonm = "Pirate"
         CFrameQuestm = CFrame.new(-1139.59717, 4.75205183, 3825.16211, -0.959730506, -7.5857054e-09, 0.280922383, -4.06310328e-08, 1, -1.11807175e-07, -0.280922383, -1.18718916e-07, -0.959730506)
         CFrameMonm = CFrame.new(-1107.5354, 13.7520456, 3969.72046)
      elseif MobsPick == "Brute [Lv. 45]"  then -- Brute
         NaemQuestm = "BuggyQuest1"
         LevelQuestm= 2
         NameMonm = "Brute"
         CFrameQuestm = CFrame.new(-1139.59717, 4.75205183, 3825.16211, -0.959730506, -7.5857054e-09, 0.280922383, -4.06310328e-08, 1, -1.11807175e-07, -0.280922383, -1.18718916e-07, -0.959730506)
         CFrameMonm = CFrame.new(-1116.89844, 14.8098745, 4279.84961)
      elseif MobsPick == "Desert Bandit [Lv. 60]" then -- Desert Bandit
         NaemQuestm = "DesertQuest"
         LevelQuestm= 1
         NameMonm = "Desert Bandit"
         CFrameQuestm = CFrame.new(897.031128, 6.43846416, 4388.97168, -0.804044724, 3.68233266e-08, 0.594568789, 6.97835176e-08, 1, 3.24365246e-08, -0.594568789, 6.75715199e-08, -0.804044724)
         CFrameMonm = CFrame.new(918.105469, 6.43846178, 4370.0249)
      elseif MobsPick == "Desert Officer [Lv. 70]" then -- Desert Officre
         NaemQuestm = "DesertQuest"
         LevelQuestm= 2
         NameMonm = "Desert Officer"
         CFrameQuestm = CFrame.new(897.031128, 6.43846416, 4388.97168, -0.804044724, 3.68233266e-08, 0.594568789, 6.97835176e-08, 1, 3.24365246e-08, -0.594568789, 6.75715199e-08, -0.804044724)
         CFrameMonm = CFrame.new(1537.75635, 14.4520388, 4378.12891)
      elseif MobsPick == "Snow Bandit [Lv. 90]" then -- Snow Bandits
         NaemQuestm = "SnowQuest"
         LevelQuestm= 1
         NameMonm = "Snow Bandits"
         CFrameQuestm = CFrame.new(1384.14001, 87.272789, -1297.06482, 0.348555952, -2.53947841e-09, -0.937287986, 1.49860568e-08, 1, 2.86358204e-09, 0.937287986, -1.50443711e-08, 0.348555952)
         CFrameMonm = CFrame.new(1389.47705, 87.272789, -1360.79626)
      elseif MobsPick == "Snowman [Lv. 100]" then -- Snowman
         NaemQuestm = "SnowQuest"
         LevelQuestm= 2
         NameMonm = "Snowman"
         CFrameQuestm = CFrame.new(1384.14001, 87.272789, -1297.06482, 0.348555952, -2.53947841e-09, -0.937287986, 1.49860568e-08, 1, 2.86358204e-09, 0.937287986, -1.50443711e-08, 0.348555952)
         CFrameMonm = CFrame.new(1234.13159, 105.777832, -1417.06055)
      elseif MobsPick == "Chief Petty Officer [Lv. 120]" then -- Chief Petty Officer
         NaemQuestm = "MarineQuest2"
         LevelQuestm= 1
         NameMonm = "Chief Petty Officer"
         CFrameQuestm = CFrame.new(-5035.0835, 28.6520386, 4325.29443, 0.0243340395, -7.08064647e-08, 0.999703884, -6.36926814e-08, 1, 7.23777944e-08, -0.999703884, -6.54350671e-08, 0.0243340395)
         CFrameMonm = CFrame.new(-4872.00049, 22.6520348, 4274.36523)
      elseif MobsPick == "Sky Bandit [Lv. 150]" then -- Sky Bandit
         NaemQuestm = "SkyQuest"
         LevelQuestm= 1
         NameMonm = "Sky Bandit"
         CFrameQuestm = CFrame.new(-4841.83447, 717.669617, -2623.96436, -0.875942111, 5.59710216e-08, -0.482416272, 3.04023082e-08, 1, 6.08195947e-08, 0.482416272, 3.86078725e-08, -0.875942111)
         CFrameMonm = CFrame.new(-4955.36279, 295.744293, -2894.04858)
      elseif MobsPick ==  "Dark Master [Lv. 175]" then -- Dark Master
         NaemQuestm = "SkyQuest"
         LevelQuestm= 2
         NameMonm = "Dark Master"
         CFrameQuestm = CFrame.new(-4841.83447, 717.669617, -2623.96436, -0.875942111, 5.59710216e-08, -0.482416272, 3.04023082e-08, 1, 6.08195947e-08, 0.482416272, 3.86078725e-08, -0.875942111)
         CFrameMonm = CFrame.new(-5247.03955, 393.063049, -2206.23926)
      elseif MobsPick == "Toga Warrior [Lv. 225]"  then -- Toga Warrior
         NaemQuestm = "ColosseumQuest"
         LevelQuestm= 1
         NameMonm = "Toga Warrior"
         CFrameQuestm = CFrame.new(-1576.11743, 7.38933945, -2983.30762, 0.576966345, 1.22114863e-09, 0.816767931, -3.58496594e-10, 1, -1.24185606e-09, -0.816767931, 4.2370063e-10, 0.576966345)
         CFrameMonm = CFrame.new(-1895.05359, 7.28907108, -2758.38428)
      elseif MobsPick == "Gladiator [Lv. 275]" then -- Gladiato
         NaemQuestm = "ColosseumQuest"
         LevelQuestm= 2
         NameMonm = "Gladiato"
         CFrameQuestm = CFrame.new(-1576.11743, 7.38933945, -2983.30762, 0.576966345, 1.22114863e-09, 0.816767931, -3.58496594e-10, 1, -1.24185606e-09, -0.816767931, 4.2370063e-10, 0.576966345)
         CFrameMonm = CFrame.new(-1270.75, 7.38933659, -3321.20752)
      elseif MobsPick == "Military Soldier [Lv. 300]" then -- Military Soldier
         NaemQuestm = "MagmaQuest"
         LevelQuestm= 1
         NameMonm = "Military Soldier"
         CFrameQuestm = CFrame.new(-5316.55859, 12.2370615, 8517.2998, 0.588437557, -1.37880001e-08, -0.808542669, -2.10116209e-08, 1, -3.23446478e-08, 0.808542669, 3.60215964e-08, 0.588437557)
         CFrameMonm = CFrame.new(-5411.63965, 11.0802879, 8454.12012)
      elseif MobsPick == "Military Spy [Lv. 330]"  then -- Military Spy
         NaemQuestm = "MagmaQuest"
         LevelQuestm= 2
         NameMonm = "Military Spy"
         CFrameQuestm = CFrame.new(-5316.55859, 12.2370615, 8517.2998, 0.588437557, -1.37880001e-08, -0.808542669, -2.10116209e-08, 1, -3.23446478e-08, 0.808542669, 3.60215964e-08, 0.588437557)
         CFrameMonm = CFrame.new(-5836.07959, 77.2306595, 8827.66211)
      elseif MobsPick == "Fishman Warrior [Lv. 375]" then -- Fishman Warrior
         NaemQuestm = "FishmanQuest"
         LevelQuestm= 1
         NameMonm = "Fishman Warrior"
         CFrameQuestm = CFrame.new(61122.5625, 18.4716396, 1568.16504, 0.893533468, 3.95251609e-09, 0.448996574, -2.34327455e-08, 1, 3.78297464e-08, -0.448996574, -4.43233645e-08, 0.893533468)
         CFrameMonm = CFrame.new(60911.8906, 18.4828224, 1585.7478)
      elseif MobsPick == "Fishman Commando [Lv. 400]" then -- Fishman Commando
         NaemQuestm = "FishmanQuest"
         LevelQuestm= 2
         NameMonm = "Fishman Commando"
         CFrameQuestm = CFrame.new(61122.5625, 18.4716396, 1568.16504, 0.893533468, 3.95251609e-09, 0.448996574, -2.34327455e-08, 1, 3.78297464e-08, -0.448996574, -4.43233645e-08, 0.893533468)
         CFrameMonm = CFrame.new(61916.8438, 18.4828224, 1444.38635)
      elseif MobsPick == "God's Guard [Lv. 450]" then -- God's Guards
         NaemQuestm = "SkyExp1Quest"
         LevelQuestm= 1
         NameMonm = "God's Guards"
         CFrameQuestm = CFrame.new(-4721.71436, 845.277161, -1954.20105, -0.999277651, -5.56969759e-09, 0.0380011722, -4.14751478e-09, 1, 3.75035256e-08, -0.0380011722, 3.73188307e-08, -0.999277651)
         CFrameMonm = CFrame.new(-4711.30566, 845.277161, -1902.31335)
      elseif MobsPick == "Shanda [Lv. 475]" then -- Shandas
         NaemQuestm = "SkyExp1Quest"
         LevelQuestm= 2
         NameMonm = "Shandas"
         CFrameQuestm = CFrame.new(-7863.63672, 5545.49316, -379.826324, 0.362120807, -1.98046344e-08, -0.93213129, 4.05822291e-08, 1, -5.48095125e-09, 0.93213129, -3.58431969e-08, 0.362120807)
         CFrameMonm = CFrame.new(-7666.93408, 5545.49316, -484.052582)
      elseif MobsPick == "Royal Squad [Lv. 525]" then -- Royal Squad
         NaemQuestm = "SkyExp2Quest"
         LevelQuestm= 1
         NameMonm = "Royal Squad"
         CFrameQuestm = CFrame.new(-7902.66895, 5635.96387, -1411.71802, 0.0504222959, 2.5710392e-08, 0.998727977, 1.12541557e-07, 1, -3.14249675e-08, -0.998727977, 1.13982921e-07, 0.0504222959)
         CFrameMonm = CFrame.new(-7684.97266, 5606.87842, -1454.46094)
      elseif MobsPick == "Royal Soldier [Lv. 550]" then -- Royal Soldier
         NaemQuestm = "SkyExp2Quest"
         LevelQuestm= 2
         NameMonm = "Royal Soldier"
         CFrameQuestm = CFrame.new(-7902.66895, 5635.96387, -1411.71802, 0.0504222959, 2.5710392e-08, 0.998727977, 1.12541557e-07, 1, -3.14249675e-08, -0.998727977, 1.13982921e-07, 0.0504222959)
         CFrameMonm = CFrame.new(-7836.83936, 5606.87842, -1745.65442)
      elseif MobsPick == "Galley Pirate [Lv. 625]" then -- Galley Pirate
         NaemQuestm = "FountainQuest"
         LevelQuestm= 1
         NameMonm = "Galley Pirate"
         CFrameQuestm = CFrame.new(5254.60156, 38.5011406, 4049.69678, -0.0504891425, -3.62066501e-08, -0.998724639, -9.87921389e-09, 1, -3.57534553e-08, 0.998724639, 8.06145284e-09, -0.0504891425)
         CFrameMonm = CFrame.new(5584.53809, 38.5386353, 3979.62256)
      elseif MobsPick == "Galley Captain [Lv. 650]" then -- Galley Captain
         NaemQuestm = "FountainQuest"
         LevelQuestm= 2
         NameMonm = "Galley Captain"
         CFrameQuestm = CFrame.new(5254.60156, 38.5011406, 4049.69678, -0.0504891425, -3.62066501e-08, -0.998724639, -9.87921389e-09, 1, -3.57534553e-08, 0.998724639, 8.06145284e-09, -0.0504891425)
         CFrameMonm = CFrame.new(5778.60547, 60.1919327, 4915.27734)
      end
   end
end
 --Check World again--
 if placeId == 2753915549 then
   FirstWorld = true
elseif placeId == 4442272183 then
   SecondWorld = true
elseif placeId == 7449423635 then
   ThirdWorld = true
end

if ThirdWorld then
   ListMob = {
      "Pirate Millionaire [Lv. 1500]";
      "Pistol Billionaire [Lv. 1525]";
      "Dragon Crew Warrior [Lv. 1575]";
      "Dragon Crew Archer [Lv. 1600]";
      "Female Islander [Lv. 1625]";
      "Giant Islander [Lv. 1650]";
      "Marine Commodore [Lv. 1700]";
      "Marine Rear Admiral [Lv. 1725]";
      "Fishman Captain [Lv. 1800]";
      "Forest Pirate [Lv. 1825]";
      "Mythological Pirate [Lv. 1850]";
      "Jungle Pirate [Lv. 1900]";
      "Musketeer Pirate [Lv. 1925]";
      "Reborn Skeleton [Lv. 1975]";
      "Living Zombie [Lv. 2000]";
      "Demonic Soul [Lv. 2025]";
      "Posessed Mummy [Lv. 2050]";
   }
elseif FirstWorld then
   ListMob = {
      "Bandit [Lv. 5]";
      "Monkey [Lv. 14]";
      "Gorilla [Lv. 20]";
      "Pirate [Lv. 35]";
      "Brute [Lv. 45]";
      "Desert Bandit [Lv. 60]";
      "Desert Officer [Lv. 70]";
      "Snow Bandit [Lv. 90]";
      "Snowman [Lv. 100]";
      "Chief Petty Officer [Lv. 120]";
      "Sky Bandit [Lv. 150]";
      "Dark Master [Lv. 175]";
      "Toga Warrior [Lv. 225]";
      "Gladiator [Lv. 275]";
      "Military Soldier [Lv. 300]";
      "Military Spy [Lv. 330]";
      "Fishman Warrior [Lv. 375]";
      "Fishman Commando [Lv. 400]";
      "God's Guard [Lv. 450]";
      "Shanda [Lv. 475]";
      "Royal Squad [Lv. 525]";
      "Royal Soldier [Lv. 550]";
      "Galley Pirate [Lv. 625]";
      "Galley Captain [Lv. 650]";
   }
elseif SecondWorld then
   ListMob = {
      "Raider [Lv. 700]";
      "Mercenary [Lv. 725]"; 
      "Swan Pirate [Lv. 775]"; 
      "Factory Staff [Lv. 800]"; 
      "Marine Lieutenant [Lv. 875]";
      "Marine Captain [Lv. 900]";
      "Zombie [Lv. 950]"; 
      "Vampire [Lv. 975]"; 
      "Snow Trooper [Lv. 1000]";
      "Winter Warrior [Lv. 1050]";
      "Lab Subordinate [Lv. 1100]";
      "Horned Warrior [Lv. 1125]";
      "Magma Ninja [Lv. 1175]";
      "Lava Pirate [Lv. 1200]"; 
      "Ship Deckhand [Lv. 1250]";
      "Ship Engineer [Lv. 1275]";
      "Ship Steward [Lv. 1300]";
      "Ship Officer [Lv. 1325]";
      "Arctic Warrior [Lv. 1350]";
      "Snow Lurker [Lv. 1375]";
      "Sea Soldier [Lv. 1425]";
      "Water Fighter [Lv. 1450]";
   }
end

ListMelee = {
   "Combat",
   "Black Leg";
   "Electro";
   "Fishman Karate";
   "Dragon Claw";
   "Superhuman";
   "DeathStep";
   "SharkmanKarate";
   "Electric Claw";
}

function CheckQuestBoss()
	if SelectBoss == "Diamond [Lv. 750] [Boss]" then
	   MsBoss = "Diamond [Lv. 750] [Boss]"
		NaemQuestBoss = "Area1Quest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-424.080078, 73.0055847, 1836.91589, 0.253544956, -1.42165932e-08, 0.967323601, -6.00147771e-08, 1, 3.04272909e-08, -0.967323601, -6.5768397e-08, 0.253544956)
		CFrameBoss = CFrame.new(-1736.26587, 198.627731, -236.412857, -0.997808516, 0, -0.0661673471, 0, 1, 0, 0.0661673471, 0, -0.997808516)
	elseif SelectBoss == "Jeremy [Lv. 850] [Boss]" then
		MsBoss = "Jeremy [Lv. 850] [Boss]"
		NaemQuestBoss = "Area2Quest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(632.698608, 73.1055908, 918.666321, -0.0319722369, 8.96074881e-10, -0.999488771, 1.36326533e-10, 1, 8.92172336e-10, 0.999488771, -1.07732087e-10, -0.0319722369)
		CFrameBoss = CFrame.new(2203.76953, 448.966034, 752.731079, -0.0217453763, 0, -0.999763548, 0, 1, 0, 0.999763548, 0, -0.0217453763)
	elseif SelectBoss == "Fajita [Lv. 925] [Boss]" then
		MsBoss = "Fajita [Lv. 925] [Boss]"
		NaemQuestBoss = "MarineQuest3"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-2442.65015, 73.0511475, -3219.11523, -0.873540044, 4.2329841e-08, -0.486752301, 5.64383384e-08, 1, -1.43220786e-08, 0.486752301, -3.99823996e-08, -0.873540044)
		CFrameBoss = CFrame.new(-2297.40332, 115.449463, -3946.53833, 0.961227536, -1.46645796e-09, -0.275756449, -2.3212845e-09, 1, -1.34094433e-08, 0.275756449, 1.35296352e-08, 0.961227536)
	elseif SelectBoss == "Don Swan [Lv. 1000] [Boss]" then
		MsBoss = "Don Swan [Lv. 1000] [Boss]"
		CFrameBoss = CFrame.new(2288.802, 15.1870775, 863.034607, 0.99974072, -8.41247214e-08, -0.0227668174, 8.4774733e-08, 1, 2.75850098e-08, 0.0227668174, -2.95079072e-08, 0.99974072)
	elseif SelectBoss == "Smoke Admiral [Lv. 1150] [Boss]" then
		MsBoss = "Smoke Admiral [Lv. 1150] [Boss]"
		NaemQuestBoss = "IceSideQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-6059.96191, 15.9868021, -4904.7373, -0.444992423, -3.0874483e-09, 0.895534337, -3.64098796e-08, 1, -1.4644522e-08, -0.895534337, -3.91229982e-08, -0.444992423)
		CFrameBoss = CFrame.new(-5115.72754, 23.7664986, -5338.2207, 0.251453817, 1.48345061e-08, -0.967869282, 4.02796978e-08, 1, 2.57916977e-08, 0.967869282, -4.54708946e-08, 0.251453817)
	elseif SelectBoss == "Cursed Captain [Lv. 1325] [Raid Boss]" then
		MsBoss = "Cursed Captain [Lv. 1325] [Raid Boss]"
		CFrameBoss = CFrame.new(916.928589, 181.092773, 33422, -0.999505103, 9.26310495e-09, 0.0314563364, 8.42916226e-09, 1, -2.6643713e-08, -0.0314563364, -2.63653774e-08, -0.999505103)
	elseif SelectBoss == "Awakened Ice Admiral [Lv. 1400] [Boss]" then
		MsBoss = "Awakened Ice Admiral [Lv. 1400] [Boss]"
		NaemQuestBoss = "FrostQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(5669.33203, 28.2118053, -6481.55908, 0.921275556, -1.25320829e-08, 0.388910472, 4.72230788e-08, 1, -7.96414241e-08, -0.388910472, 9.17372489e-08, 0.921275556)
		CFrameBoss = CFrame.new(6407.33936, 340.223785, -6892.521, 0.49051559, -5.25310213e-08, -0.871432424, -2.76146022e-08, 1, -7.58250565e-08, 0.871432424, 6.12576301e-08, 0.49051559)
	elseif SelectBoss == "Tide Keeper [Lv. 1475] [Boss]" then
		MsBoss = "Tide Keeper [Lv. 1475] [Boss]"
		NaemQuestBoss = "ForgottenQuest"             
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-3053.89648, 236.881363, -10148.2324, -0.985987961, -3.58504737e-09, 0.16681771, -3.07832915e-09, 1, 3.29612559e-09, -0.16681771, 2.73641976e-09, -0.985987961)
		CFrameBoss = CFrame.new(-3570.18652, 123.328949, -11555.9072, 0.465199202, -1.3857326e-08, 0.885206044, 4.0332897e-09, 1, 1.35347511e-08, -0.885206044, -2.72606271e-09, 0.465199202)
		-- Old World
	elseif SelectBoss == "Saber Expert [Lv. 200] [Boss]" then
		MsBoss = "Saber Expert [Lv. 200] [Boss]"
		CFrameBoss = CFrame.new(-1458.89502, 29.8870335, -50.633564, 0.858821094, 1.13848939e-08, 0.512275636, -4.85649254e-09, 1, -1.40823326e-08, -0.512275636, 9.6063415e-09, 0.858821094)
	elseif SelectBoss == "The Gorilla King [Lv. 25] [Boss]" then
		MsBoss = "The Gorilla King [Lv. 25] [Boss]"
		NaemQuestBoss = "JungleQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-1604.12012, 36.8521118, 154.23732, 0.0648873374, -4.70858913e-06, -0.997892559, 1.41431883e-07, 1, -4.70933674e-06, 0.997892559, 1.64442184e-07, 0.0648873374)
		CFrameBoss = CFrame.new(-1223.52808, 6.27936459, -502.292664, 0.310949147, -5.66602516e-08, 0.950426519, -3.37275488e-08, 1, 7.06501808e-08, -0.950426519, -5.40241736e-08, 0.310949147)
	elseif SelectBoss == "Bobby [Lv. 55] [Boss]" then
		MsBoss = "Bobby [Lv. 55] [Boss]"
		NaemQuestBoss = "BuggyQuest1"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-1139.59717, 4.75205183, 3825.16211, -0.959730506, -7.5857054e-09, 0.280922383, -4.06310328e-08, 1, -1.11807175e-07, -0.280922383, -1.18718916e-07, -0.959730506)
		CFrameBoss = CFrame.new(-1147.65173, 32.5966301, 4156.02588, 0.956680477, -1.77109952e-10, -0.29113996, 5.16530874e-10, 1, 1.08897802e-09, 0.29113996, -1.19218679e-09, 0.956680477)
	elseif SelectBoss == "Yeti [Lv. 110] [Boss]" then
		MsBoss = "Yeti [Lv. 110] [Boss]"
		NaemQuestBoss = "SnowQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(1384.90247, 87.3078308, -1296.6825, 0.280209213, 2.72035177e-08, -0.959938943, -6.75690828e-08, 1, 8.6151708e-09, 0.959938943, 6.24481444e-08, 0.280209213)
		CFrameBoss = CFrame.new(1221.7356, 138.046906, -1488.84082, 0.349343032, -9.49245944e-08, 0.936994851, 6.29478194e-08, 1, 7.7838429e-08, -0.936994851, 3.17894653e-08, 0.349343032)
	elseif SelectBoss == "Mob Leader [Lv. 120] [Boss]" then
		MsBoss = "Mob Leader [Lv. 120] [Boss]"
		CFrameBoss = CFrame.new(-2848.59399, 7.4272871, 5342.44043, -0.928248107, -8.7248246e-08, 0.371961564, -7.61816636e-08, 1, 4.44474857e-08, -0.371961564, 1.29216433e-08, -0.928248107)
		--The Gorilla King [Lv. 25] [Boss]
	elseif SelectBoss == "Vice Admiral [Lv. 130] [Boss]" then
		MsBoss = "Vice Admiral [Lv. 130] [Boss]"
		NaemQuestBoss = "MarineQuest2"
		LevelQuestBoss = 2
		CFrameQuestBoss = CFrame.new(-5035.42285, 28.6520386, 4324.50293, -0.0611100644, -8.08395768e-08, 0.998130739, -1.57416586e-08, 1, 8.00271849e-08, -0.998130739, -1.08217701e-08, -0.0611100644)
		CFrameBoss = CFrame.new(-5078.45898, 99.6520691, 4402.1665, -0.555574954, -9.88630566e-11, 0.831466436, -6.35508286e-08, 1, -4.23449258e-08, -0.831466436, -7.63661632e-08, -0.555574954)
	elseif SelectBoss == "Warden [Lv. 175] [Boss]" then
		MsBoss = "Warden [Lv. 175] [Boss]"
		NaemQuestBoss = "ImpelQuest"
		LevelQuestBoss = 1
		CFrameQuestBoss = CFrame.new(4851.35059, 5.68744135, 743.251282, -0.538484037, -6.68303741e-08, -0.842635691, 1.38001752e-08, 1, -8.81300792e-08, 0.842635691, -5.90851599e-08, -0.538484037)
		CFrameBoss = CFrame.new(5232.5625, 5.26856995, 747.506897, 0.943829298, -4.5439414e-08, 0.330433697, 3.47818627e-08, 1, 3.81658154e-08, -0.330433697, -2.45289105e-08, 0.943829298)
	elseif SelectBoss == "Chief Warden [Lv. 200] [Boss]" then
		MsBoss = "Chief Warden [Lv. 200] [Boss]"
		NaemQuestBoss = "ImpelQuest"
		LevelQuestBoss = 2
		CFrameQuestBoss = CFrame.new(4851.35059, 5.68744135, 743.251282, -0.538484037, -6.68303741e-08, -0.842635691, 1.38001752e-08, 1, -8.81300792e-08, 0.842635691, -5.90851599e-08, -0.538484037)
		CFrameBoss = CFrame.new(5232.5625, 5.26856995, 747.506897, 0.943829298, -4.5439414e-08, 0.330433697, 3.47818627e-08, 1, 3.81658154e-08, -0.330433697, -2.45289105e-08, 0.943829298)
	elseif SelectBoss == "Flamingo [Lv. 225] [Boss]" then
		MsBoss = "Flamingo [Lv. 225] [Boss]"
		NaemQuestBoss = "ImpelQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(4851.35059, 5.68744135, 743.251282, -0.538484037, -6.68303741e-08, -0.842635691, 1.38001752e-08, 1, -8.81300792e-08, 0.842635691, -5.90851599e-08, -0.538484037)
		CFrameBoss = CFrame.new(5232.5625, 5.26856995, 747.506897, 0.943829298, -4.5439414e-08, 0.330433697, 3.47818627e-08, 1, 3.81658154e-08, -0.330433697, -2.45289105e-08, 0.943829298)
	elseif SelectBoss == "Magma Admiral [Lv. 350] [Boss]" then
		MsBoss = "Magma Admiral [Lv. 350] [Boss]"
		NaemQuestBoss = "MagmaQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-5317.07666, 12.2721891, 8517.41699, 0.51175487, -2.65508806e-08, -0.859131515, -3.91131572e-08, 1, -5.42026761e-08, 0.859131515, 6.13418294e-08, 0.51175487)
		CFrameBoss = CFrame.new(-5530.12646, 22.8769703, 8859.91309, 0.857838571, 2.23414389e-08, 0.513919294, 1.53689133e-08, 1, -6.91265853e-08, -0.513919294, 6.71978384e-08, 0.857838571)
	elseif SelectBoss == "Fishman Lord [Lv. 425] [Boss]" then
		MsBoss = "Fishman Lord [Lv. 425] [Boss]"
		NaemQuestBoss = "FishmanQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(61123.0859, 18.5066795, 1570.18018, 0.927145958, 1.0624845e-07, 0.374700129, -6.98219367e-08, 1, -1.10790765e-07, -0.374700129, 7.65569368e-08, 0.927145958)
		CFrameBoss = CFrame.new(61351.7773, 31.0306778, 1113.31409, 0.999974668, 0, -0.00714713801, 0, 1.00000012, 0, 0.00714714266, 0, 0.999974549)
	elseif SelectBoss == "Wysper [Lv. 500] [Boss]" then
		MsBoss = "Wysper [Lv. 500] [Boss]"
		NaemQuestBoss = "SkyExp1Quest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-7862.94629, 5545.52832, -379.833954, 0.462944925, 1.45838088e-08, -0.886386991, 1.0534996e-08, 1, 2.19553424e-08, 0.886386991, -1.95022007e-08, 0.462944925)
		CFrameBoss = CFrame.new(-7925.48389, 5550.76074, -636.178345, 0.716468513, -1.22915289e-09, 0.697619379, 3.37381434e-09, 1, -1.70304748e-09, -0.697619379, 3.57381835e-09, 0.716468513)
	elseif SelectBoss == "Thunder God [Lv. 575] [Boss]" then
		MsBoss = "Thunder God [Lv. 575] [Boss]"
		NaemQuestBoss = "SkyExp2Quest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(-7902.78613, 5635.99902, -1411.98706, -0.0361216255, -1.16895912e-07, 0.999347389, 1.44533963e-09, 1, 1.17024491e-07, -0.999347389, 5.6715117e-09, -0.0361216255)
		CFrameBoss = CFrame.new(-7917.53613, 5616.61377, -2277.78564, 0.965189934, 4.80563429e-08, -0.261550069, -6.73089886e-08, 1, -6.46515304e-08, 0.261550069, 8.00056768e-08, 0.965189934)
	elseif SelectBoss == "Cyborg [Lv. 675] [Boss]" then
		MsBoss = "Cyborg [Lv. 675] [Boss]"
		NaemQuestBoss = "FountainQuest"
		LevelQuestBoss = 3
		CFrameQuestBoss = CFrame.new(5253.54834, 38.5361786, 4050.45166, -0.0112687312, -9.93677887e-08, -0.999936521, 2.55291371e-10, 1, -9.93769547e-08, 0.999936521, -1.37512213e-09, -0.0112687312)
		CFrameBoss = CFrame.new(6041.82813, 52.7112198, 3907.45142, -0.563162148, 1.73805248e-09, -0.826346457, -5.94632716e-08, 1, 4.26280238e-08, 0.826346457, 7.31437524e-08, -0.563162148)
   elseif SelectBoss == "Stone [Lv. 1550] [Boss]" then
      MsBoss = "Stone [Lv. 1550] [Boss]"
      NaemQuestBoss = "PiratePortQuest"
      LevelQuestBoss = 3
      CFrameQuestBoss = CFrame.new(-288.003815, 43.7675667, 5573.12012, -0.959750533, 1.11968195e-08, 0.280853927, -1.77952231e-08, 1, -1.00677937e-07, -0.280853927, -1.01623563e-07, -0.959750533)
      CFrameBoss = CFrame.new(-1086.11621, 38.8425903, 6768.71436, 0.0231462717, -0.592676699, 0.805107772, 2.03251839e-05, 0.805323839, 0.592835128, -0.999732077, -0.0137055516, 0.0186523199)
   elseif SelectBoss == "Island Empress [Lv. 1675] [Boss]" then
      MsBoss = "Island Empress [Lv. 1675] [Boss]"
      NaemQuestBoss = "AmazonQuest2"
      LevelQuestBoss = 3
      CFrameQuestBoss = CFrame.new(5444.14355, 601.603821, 751.306763, 0.0994469747, 6.9440496e-08, -0.995042861, -6.87854964e-08, 1, 6.29118517e-08, 0.995042861, 6.21881213e-08, 0.0994469747)
      CFrameBoss = CFrame.new(5713.98877, 601.922974, 202.751251, -0.101080291, -0, -0.994878292, -0, 1, -0, 0.994878292, 0, -0.101080291)
   elseif SelectBoss == "Kilo Admiral [Lv. 1750] [Boss]" then
      MsBoss = "Kilo Admiral [Lv. 1750] [Boss]"
      NaemQuestBoss = "MarineTreeIsland"
      LevelQuestBoss = 3
      CFrameQuestBoss = CFrame.new(2223.3645, 28.7049141, -6719.18408, -0.953810513, 0.000124398866, 0.30040893, 0.000600414874, 0.999998689, 0.00149224349, -0.300408363, 0.00160368741, -0.953809321)
      CFrameBoss = CFrame.new(2877.61743, 423.558685, -7207.31006, -0.989591599, -0, -0.143904909, -0, 1.00000012, -0, 0.143904924, 0, -0.989591479)
   elseif SelectBoss == "Captain Elephant [Lv. 1875] [Boss]" then
      MsBoss = "Captain Elephant [Lv. 1875] [Boss]"
      NaemQuestBoss = "DeepForestIsland"
      LevelQuestBoss = 3
      CFrameQuestBoss = CFrame.new(-13231.1602, 333.744446, -7624.40723, -0.0901302397, 8.36351219e-08, 0.995930016, 2.76566414e-08, 1, -8.14740204e-08, -0.995930016, 2.02008046e-08, -0.0901302397)
      CFrameBoss = CFrame.new(-13485.0283, 331.709259, -8012.4873, 0.714521289, 7.98849911e-08, 0.69961375, -1.02065748e-07, 1, -9.94383065e-09, -0.69961375, -6.43015241e-08, 0.714521289)
	end 
end
if game.PlaceId == 7449423635 or game.PlaceId == 4442272183 then
   if not getgenv().ColorNotifier then
       spawn(function()
           getgenv().ColorNotifier = true
           loadstring(game:HttpGet("https://raw.githubusercontent.com/vinhuchi/BloxFruit/main/HakiNotifier.lua"))()
           while wait(300) do
               local args = {[1] = "ColorsDealer",[2] = "1"}
               local CurrentHaki = game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               if (OldHaki == nil or OldHaki == "") or (OldHaki~=CurrentHaki) then
                   loadstring(game:HttpGet("https://raw.githubusercontent.com/vinhuchi/BloxFruit/main/HakiNotifier.lua"))()
                   local args = {[1] = "ColorsDealer",[2] = "1"}
                   OldHaki =  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               end
           end
       end)
   end
end
local PlaceID = game.PlaceId
local AllIDs = {}
local foundAnything = ""
local actualHour = os.date("!*t").hour
local Deleted = false
local File = pcall(function()
  AllIDs = game:GetService('HttpService'):JSONDecode(readfile("NotSameServers.json"))
end)

if not File then
  table.insert(AllIDs, actualHour)
  writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
end

function TPReturner()
   local Site;
   if foundAnything == "" then
      Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
   else
      Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
   end
   local ID = ""
   if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
      foundAnything = Site.nextPageCursor
   end
   local num = 0;
   for i,v in pairs(Site.data) do
      local Possible = true
      ID = tostring(v.id)
      if tonumber(v.maxPlayers) > tonumber(v.playing) then
         for _,Existing in pairs(AllIDs) do
            if num ~= 0 then
               if ID == tostring(Existing) then
                  Possible = false
               end
            else
               if tonumber(actualHour) ~= tonumber(Existing) then
                  local delFile = pcall(function()
                  delfile("NotSameServers.json")
                  AllIDs = {}
                  table.insert(AllIDs, actualHour)
                  end)
               end
            end
            num = num + 1
         end
         if Possible == true then
            table.insert(AllIDs, ID)
            wait()
            pcall(function()
               writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
               wait()
               game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
            end)
            wait(4)
         end
      end
   end
end

function Teleport()
   while game:GetService("RunService").RenderStepped:wait() do
      pcall(function()
         TPReturner()
         if foundAnything ~= "" then
            TPReturner()
         end
      end)
   end
end

CheckQuest()
CheckQuest2()
CheckQuestBoss()
SelectToolWeapon = ""

local TweenSpeed= 300
local TweenSpeedF = 205
function TeleportTween(dist, Speed)
   local char = game.Players.LocalPlayer.Character
   local hm = char:WaitForChild("HumanoidRootPart")
    local info = TweenInfo.new((hm.Position - dist.Position).magnitude / Speed,Enum.EasingStyle.Linear)
    local tween =  game:service"TweenService":Create(hm, info, {CFrame = dist})
    tween:Play()
    tween.Completed:Wait()
end

function EquipWeapon(ToolSe)
   if game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe) then
      local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe)
      wait(.4)
      game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
   end
end

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      pcall(function()
         for i, v in pairs(game:GetService("ReplicatedStorage"):GetChildren()) do
            if v:FindFirstChild("Humanoid") ~= nil  and v:FindFirstChild("HumanoidRootPart") ~= nil and v:IsA("Model") then
               v.Parent = game:GetService("Workspace").Enemies
            end
         end
         for i, v in pairs(game:GetService("Workspace").Boats:GetChildren()) do
            if v:FindFirstChild("Humanoid") ~= nil  and v:FindFirstChild("HumanoidRootPart") ~= nil and v:IsA("Model") then
               v.Parent = game:GetService("Workspace").Enemies
            end
         end
      end)
   end
end)

local lib = library.new("Know Hub")
lib:setTheme("TextColor",TextColorChange)
wait(1)

local AutoFarmPage = lib:addPage("Auto Farm", 6035145364)
local AutoFarm = AutoFarmPage:addSection("Auto Farm")

AutoFarm:addToggle("Auto Farm",false, function(x)     
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AbandonQuest")
    AF = x
    if AF then
      wait(180)
      if AF then
         AF = false
         wait(1)
         AF = true
         print("Reset af")
      end
   end
    CheckQuest()
end)

AutoFarm:addToggle("Auto Farm (Choose mobs)",false, function(x)     
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AbandonQuest")
    AFM = x
   if AFM then
      wait(180)
      if AFM then
         AFM = false
         wait(1)
         AFM = true
         print("Reset afm")
      end
   end
    CheckQuest2()
end)

local SelectMobs = AutoFarm:addDropdown("Choose mobs",ListMob,function(Value)
   MobsPick = Value
end)

if ThirdWorld then
   AutoFarm:addToggle("Auto RainbowHaki",false,function(t)
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AbandonQuest")
      RainbowHaki = t
      while RainbowHaki do
         wait()
         if game:GetService("Workspace").Enemies:FindFirstChild("Stone [Lv. 1550] [Boss]") ~= nil or game:GetService("Workspace").Enemies:FindFirstChild("Island Empress [Lv. 1675] [Boss]") ~= nil 
         or game:GetService("Workspace").Enemies:FindFirstChild("Kilo Admiral [Lv. 1750] [Boss]") ~= nil or  game:GetService("Workspace").Enemies:FindFirstChild("Captain Elephant [Lv. 1875] [Boss]") ~= nil
         or game:GetService("Workspace").Enemies:FindFirstChild("Beautiful Pirate [Lv. 1950] [Boss]") ~= nil then
            if game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible == false then
           --    TeleportTween(CFrame.new(-11891.202148438, 930.57678222656, -8760.0498046875),TweenSpeed)
               wait(.5)
               game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("HornedMan")
               game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("HornedMan","Bet")
            else
               if game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text == "Defeat  Stone (0/1)" then
                  if game:GetService("Workspace").Enemies:FindFirstChild("Stone [Lv. 1550] [Boss]") then
                     for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                        if v.Name == "Stone [Lv. 1550] [Boss]" then
                           if not game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
                              local args = {
                                 [1] = "Buso"
                              }
                              game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                           end
                           EquipWeapon(SelectToolWeapon)
                           TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,17,0),TweenSpeed)
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                           game:GetService'VirtualUser':CaptureController()
                           game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                        end
                     end
                  else
                     TeleportTween(CFrame.new(-1109.6944580078, 40.006885528564, 6730.9833984375),TweenSpeed)
                  end
               elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text == "Defeat  Island Empress (0/1)" then
                  if game:GetService("Workspace").Enemies:FindFirstChild("Island Empress [Lv. 1675] [Boss]") then
                     for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                        if v.Name == "Island Empress [Lv. 1675] [Boss]" then
                           if not game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
                              local args = {
                                 [1] = "Buso"
                              }
                              game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                           end
                           EquipWeapon(SelectToolWeapon)
                           TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,17,0),TweenSpeed)
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                           game:GetService'VirtualUser':CaptureController()
                           game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                        end
                     end
                  else
                     TeleportTween(CFrame.new(5702.2724609375, 601.94860839844, 201.07873535156),TweenSpeed)
                  end
               elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text == "Defeat  Kilo Admiral (0/1)" then
                  if game:GetService("Workspace").Enemies:FindFirstChild("Kilo Admiral [Lv. 1750] [Boss]") then
                     for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                        if v.Name == "Kilo Admiral [Lv. 1750] [Boss]" then
                           if not game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
                              local args = {
                                 [1] = "Buso"
                              }
                              game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                           end
                           EquipWeapon(SelectToolWeapon)
                           TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,17,0),TweenSpeed)
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                           game:GetService'VirtualUser':CaptureController()
                           game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                        end
                     end
                  else
                     TeleportTween(CFrame.new(2861.53515625, 423.58441162109, -7254.0751953125),TweenSpeed)
                  end
               elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text == "Defeat  Captain Elephant (0/1)" then
                  if game:GetService("Workspace").Enemies:FindFirstChild("Captain Elephant [Lv. 1875] [Boss]") then
                     for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                        if v.Name == "Captain Elephant [Lv. 1875] [Boss]" then
                           if not game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
                              local args = {
                                 [1] = "Buso"
                              }
                              game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                           end
                           EquipWeapon(SelectToolWeapon)
                           TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,17,0),TweenSpeed)
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                           game:GetService'VirtualUser':CaptureController()
                           game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                        end
                     end
                  else
                     TeleportTween(CFrame.new(-13493.12890625, 318.89553833008, -8373.7919921875),TweenSpeed)
                  end
               elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text == "Defeat  Beautiful Pirate (0/1)" then
                  if game:GetService("Workspace").Enemies:FindFirstChild("Beautiful Pirate [Lv. 1950] [Boss]") then
                     for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                        if v.Name == "Beautiful Pirate [Lv. 1950] [Boss]" then
                           if not game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
                              local args = {
                                 [1] = "Buso"
                              }
                              game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                           end
                           EquipWeapon(SelectToolWeapon)
                           TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,17,0),TweenSpeed)
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(50,50,50)
                           game:GetService'VirtualUser':CaptureController()
                           game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                        end
                     end
                  else
                     TeleportTween(CFrame.new(5378.6337890625, 22.56223487854, -26.053804397583),TweenSpeed)
                  end
               end
            end
         end
      end
   end)
end

AutoFarm:addToggle("Auto Set Spawn (Auto Farm)",false, function(x)
   AutoSetSpawn = x
end)

AutoFarm:addSlider("AutoFarm Distance",getgenv().Distance,1,30,function(p)
    getgenv().Distance = p
end)

function AutoSecondworld()
   while AutoSecondWorld do
      wait(0.1)
      local MyLevel = game.Players.localPlayer.Data.Level.Value
      if MyLevel >= 700 and FirstWorld then
         if AF == true then
            AF = false
            ChangeAF = true
         end
         if AFM == true then
            AFM = false
            ChangeAFM = true
         end
         SelectToolWeapon = "Key"
         TeleportTween(CFrame.new(4849.29883, 5.65138149, 719.611877),TweenSpeed)
         wait(0.5)
         local args = {
            [1] = "DressrosaQuestProgress",
            [2] = "Detective"
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
         wait(0.5)
         if game.Players.LocalPlayer.Backpack:FindFirstChild("Key") then
            local tool = game.Players.LocalPlayer.Backpack:FindFirstChild("Key")
            wait(.4)
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
         end
         TeleportTween(CFrame.new(1347.7124, 37.3751602, -1325.6488),TweenSpeed)
         wait(0.5)
         function click()
            game:GetService'VirtualUser':CaptureController()
            game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
         end
         game.Players.LocalPlayer.Character.Humanoid:UnequipTools()
         for i,v in pairs(ListMelee) do
            if game.Players.LocalPlayer.Backpack:FindFirstChild(v) ~= nil and game.Players.LocalPlayer.Character:FindFirstChild(v) == nil then
               EquipWeapon(v)
            end
         end
         if game.Workspace.Enemies:FindFirstChild("Ice Admiral [Lv. 700] [Boss]") and game.Workspace.Map.Ice.Door.CanCollide == false and game.Workspace.Map.Ice.Door.Transparency == 1 then
            CheckBoss = true
            for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
               if CheckBoss and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and v.Name == "Ice Admiral [Lv. 700] [Boss]" then
                  repeat wait(.1)
                     EquipWeapon(SelectToolWeapon)
                     pcall(function()
                        v.HumanoidRootPart.Transparency = 0.5
                        v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                        v.HumanoidRootPart.BrickColor = BrickColor.new("White")
                        v.HumanoidRootPart.CanCollide = false
                        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame*CFrame.new(0, 25, 0)
                        click()
                     end)
                  until not CheckBoss or not v.Parent or v.Humanoid.Health <= 0 or AutoSecondWorld == false
               end
            end
            CheckBoss = false
            wait(0.5)
            TeleportTween(CFrame.new(-1166.23743, 7.65220165, 1728.36487),TweenSpeed)
            wait(0.5)
            local args = {
               [1] = "TravelDressrosa" -- OLD WORLD to NEW WORLD
            }
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
         else
            if game.Players.LocalPlayer.Backpack:FindFirstChild("Key") then
               local tool = game.Players.LocalPlayer.Backpack:FindFirstChild("Key")
               wait(.4)
               game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
            end
         TeleportTween(CFrame.new(1347.7124, 37.3751602, -1325.6488),TweenSpeed)
         end
      end
   end
end

if FirstWorld then
   AutoFarm:addToggle("Auto Sec World",getgenv().AutoSecondWorld, function(x)
      AutoSecondWorld = x
      AutoSecondworld()
   end)
end

function AutoThirdWorld()
   while AutoThirdworld do
      wait()
      local MyLevel = game.Players.localPlayer.Data.Level.Value
      if MyLevel >= 1500 and SecondWorld then
         if AF == true then
            AF = false
            ChangeAF = true
         end
         if AFM == true then
            AFM = false
            ChangeAFM = true
         end
         wait(3)
         if game.Workspace.Enemies:FindFirstChild("rip_indra [Lv. 1500] [Boss]") then
            CheckRIP = true
            for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
               if CheckRIP and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and v.Name == "rip_indra [Lv. 1500] [Boss]" then
                  Hit = true
                  TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeed)
                  repeat wait(.1)
                     count = 0
                     EquipWeapon(SelectToolWeapon)
                     pcall(function()
                        v.HumanoidRootPart.Transparency = 0.5
                        v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                        v.HumanoidRootPart.BrickColor = BrickColor.new("White")
                        v.HumanoidRootPart.CanCollide = false
                        if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance  then
                           game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame*CFrame.new(0, 25, 0)
                        end
                     end)
                  until not CheckRIP or not v.Parent or v.Humanoid.Health <= 55000 or AutoThirdworld == false
                  Hit = false
               end
            end
            CheckRIP = false
            wait(0.5)
            for i = 1,3 do wait()
               game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("TravelZou")
            end
         else
            wait(1)
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("TravelZou")
         end
      end
   end
end

if SecondWorld then
   AutoFarm:addToggle("Auto Third World",getgenv().AutoThirdworld, function(x)
      AutoThirdworld = x
      AutoThirdWorld()
   end)
end


if FirstWorld then
   AutoFarm:addToggle("Auto Saber",getgenv().Saber, function(x)
      ASaber = x
      while ASaber and FirstWorld  and MyLevel >= 200  do
         wait()
         local MyLevel = game.Players.localPlayer.Data.Level.Value
         local Sword = game:GetService("Players").LocalPlayer.Data.Stats.Sword.Level.Value
         autosaber()
      end
   end)
end

function AutoRengokus()
   while AutoRengoku do
      wait()
      if game.Players.LocalPlayer.Backpack:FindFirstChild("Hidden Key") or game.Players.LocalPlayer.Character:FindFirstChild("Hidden Key") then
      
         if AF == true then
            AF = false
            ChangeAF = true
         end
         if AFM == true then
            AFM = false
            ChangeAFM = true
         end


         local toolname = "Hidden Key"
         local Plr = game:GetService("Players").LocalPlayer

            for i = 1,1 do
               pcall(function()
                  if Plr.Backpack:FindFirstChild(toolname) and Plr.Character:FindFirstChild(toolname) == nil then
                     local tool = Plr.Backpack:FindFirstChild(toolname)
                     Plr.Character.Humanoid:EquipTool(tool)
                  end
               end)
            end

         wait(0.5)
         local vChest = game:GetService("Workspace").Map.IceCastle.RengokuChest.Detection
         vChest.CanCollide = false
         vChest.Size = Vector3.new(15,15,15)
         TeleportTween(CFrame.new(6574.68555, 296.654755, -6962.2124),TweenSpeed)

         wait(1)
         AutoRengoku = false

         if ChangeAF == true then
            AF = true
            ChangeAF = false
         end
         if ChangeAFM == true then
            AFM = true
            ChangeAFM = false
         end

      end
   end
end

if SecondWorld then
   AutoFarm:addToggle("Auto Rengoku",getgenv().Rengoku, function(x)
      AutoRengoku = x
      AutoRengokus()
   end)
end


function AutoSuperHuman()
   while Superhuman do
      wait(0.5)
      pcall(function()
         if SecondWorld or ThirdWorld then
            if game.Players.LocalPlayer.Backpack:FindFirstChild("Combat") or game.Players.LocalPlayer.Character:FindFirstChild("Combat") then
               local args = {
               [1] = "BuyBlackLeg"
               }
               game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
            end
            if game.Players.LocalPlayer.Backpack:FindFirstChild("Black Leg") or game.Players.LocalPlayer.Character:FindFirstChild("Black Leg") or game.Players.LocalPlayer.Backpack:FindFirstChild("Electro") or game.Players.LocalPlayer.Character:FindFirstChild("Electro") or game.Players.LocalPlayer.Backpack:FindFirstChild("Fishman Karate") or game.Players.LocalPlayer.Character:FindFirstChild("Fishman Karate") or game.Players.LocalPlayer.Backpack:FindFirstChild("Dragon Claw") or game.Players.LocalPlayer.Character:FindFirstChild("Dragon Claw") or game.Players.LocalPlayer.Backpack:FindFirstChild("Superhuman") or game.Players.LocalPlayer.Character:FindFirstChild("Superhuman") then

               if game.Players.LocalPlayer.Backpack:FindFirstChild("Black Leg") and game.Players.LocalPlayer.Backpack:FindFirstChild("Black Leg").Level.Value <= 299 then
                  SelectToolWeapon = "Black Leg"
               end
               if game.Players.LocalPlayer.Backpack:FindFirstChild("Electro") and game.Players.LocalPlayer.Backpack:FindFirstChild("Electro").Level.Value <= 299 then
                  SelectToolWeapon = "Electro"
               end

               if game.Players.LocalPlayer.Backpack:FindFirstChild("Fishman Karate") and game.Players.LocalPlayer.Backpack:FindFirstChild("Fishman Karate").Level.Value <= 299 then
                  SelectToolWeapon = "Fishman Karate"
               end
               if game.Players.LocalPlayer.Backpack:FindFirstChild("Dragon Claw") and game.Players.LocalPlayer.Backpack:FindFirstChild("Dragon Claw").Level.Value <= 299 then
                  SelectToolWeapon = "Dragon Claw"
               end

               if game.Players.LocalPlayer.Backpack:FindFirstChild("Black Leg") and game.Players.LocalPlayer.Backpack:FindFirstChild("Black Leg").Level.Value >= 300 then
                  local args = {
                  [1] = "BuyElectro"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               end
               if game.Players.LocalPlayer.Character:FindFirstChild("Black Leg") and game.Players.LocalPlayer.Character:FindFirstChild("Black Leg").Level.Value >= 300 then
                  local args = {
                  [1] = "BuyElectro"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               end

               if game.Players.LocalPlayer.Backpack:FindFirstChild("Electro") and game.Players.LocalPlayer.Backpack:FindFirstChild("Electro").Level.Value >= 300 then
                  local args = {
                  [1] = "BuyFishmanKarate"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               end
               if game.Players.LocalPlayer.Character:FindFirstChild("Electro") and game.Players.LocalPlayer.Character:FindFirstChild("Electro").Level.Value >= 300 then
                  local args = {
                  [1] = "BuyFishmanKarate"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               end
   
               if game.Players.LocalPlayer.Backpack:FindFirstChild("Fishman Karate") and game.Players.LocalPlayer.Backpack:FindFirstChild("Fishman Karate").Level.Value >= 300 then
                  local args = {
                     [1] = "BlackbeardReward",
                     [2] = "DragonClaw",
                     [3] = "1"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                  local args = {
                     [1] = "BlackbeardReward",
                     [2] = "DragonClaw",
                     [3] = "2"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args)) 
               end

               if game.Players.LocalPlayer.Character:FindFirstChild("Fishman Karate") and game.Players.LocalPlayer.Character:FindFirstChild("Fishman Karate").Level.Value >= 300 then
                  local args = {
                     [1] = "BlackbeardReward",
                     [2] = "DragonClaw",
                     [3] = "1"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                  local args = {
                     [1] = "BlackbeardReward",
                     [2] = "DragonClaw",
                     [3] = "2"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args)) 
               end

               if game.Players.LocalPlayer.Backpack:FindFirstChild("Dragon Claw") and game.Players.LocalPlayer.Backpack:FindFirstChild("Dragon Claw").Level.Value >= 300 then
                  local args = {
                  [1] = "BuySuperhuman"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                  SelectToolWeapon = "Superhuman"
               end

               if game.Players.LocalPlayer.Character:FindFirstChild("Dragon Claw") and game.Players.LocalPlayer.Character:FindFirstChild("Dragon Claw").Level.Value >= 300 then
                  local args = {
                  [1] = "BuySuperhuman"
                  }
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                  SelectToolWeapon = "Superhuman"
               end

               if game.Players.LocalPlayer.Backpack:FindFirstChild("Superhuman") and game.Players.LocalPlayer.Backpack:FindFirstChild("Superhuman").Level.Value <= 600 then
                  SelectToolWeapon = "Superhuman"
               end
               if game.Players.LocalPlayer.Character:FindFirstChild("Superhuman") and game.Players.LocalPlayer.Character:FindFirstChild("Superhuman").Level.Value <= 600 then
                  SelectToolWeapon = "Superhuman"
               end
            end
         end
      end)
   end
end
if SecondWorld or ThirdWorld then
   AutoFarm:addToggle("Auto Superhuman",getgenv().AutoSuperhuman, function(x)
      Superhuman = x
      AutoSuperHuman()
   end)
end

if SecondWorld then
   AutoFarm:addToggle("Auto Bartilo",false, function(x)
      BartiloQ = x
      local args = {
			[1] = "getInventoryWeapons"
		}
      for i,v in pairs(game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))) do
         for i2,v2 in pairs(v) do
            if v2 == "Warrior Helmet" then
               lib:Notify("Bartilo Quest","You already completed")
               Bartilo = false
               WarriotHelmet = true
            end
         end
      end
      if game.Players.LocalPlayer.Backpack:FindFirstChild("Warrior Helmet") ~= nil or game.Players.LocalPlayer.Character:FindFirstChild("Warrior Helmet") ~= nil then
         lib:Notify("Bartilo Quest","You already completed")
         Bartilo = false
         WarriotHelmet = true
      end
      while BartiloQ  do
         game:GetService("RunService").RenderStepped:wait()
         local MyLevel = game.Players.localPlayer.Data.Level.Value
         if MyLevel >= 850 and SecondWorld and not WarriotHelmet then
            if AF == true then
               AF = false
               ChangeAF = true
            end
            if AFM == true then
               AFM = false
               ChangeAFM = true
            end
            wait(3)
            if string.find(LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text, NameMon) then
               LocalPlayer.PlayerGui.Main.Quest.Visible = false
            end
            BartiloQuest()
            if LocalPlayer.PlayerGui.Main.Quest.Visible == false then
               TeleportTween(CFrameQuestx,TweenSpeed)
               wait(1.1)
               game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", QuestName, QuestLevel)
               wait(1)
            elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible == true then
               if string.find(LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text, MobName) then
                  for i, v in pairs(game.Workspace.Enemies:GetChildren()) do
                     if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") then
                        if v.Name == Mobx then
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(60,60,60)
                           zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
                           zeroGrav(v.HumanoidRootPart)
                           PosBartilo = v.HumanoidRootPart.CFrame
                           Hit = true
                           MagBartilo = true
                           if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                              Tweening = false
                           elseif not Tweening then
                              Tweening = true
                              TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeedF)
                              Tweening = false
                           end
                           repeat wait()
                              EquipWeapon(SelectToolWeapon)
                              if Mode == "Magnet" then
                                 if sethiddenproperty then 
                                    sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                                 end
                              end
                              MagBartilo = true
                              if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                                 game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 25, 0)
                              elseif not Tweening then
                                 Tweening = true
                                 TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeedF)
                                 Tweening = false
                              end
                           until v.Humanoid.Health <= 0 or BartiloQ == false
                           Hit = false
                           MagBartilo = false
                        end
                     end
                  end
               else game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible = false 
                  print("Bartilo Success")
               end
               if LocalPlayer.PlayerGui.Main.Quest.Visible == false then
                  for i, v in pairs(game.Workspace.Enemies:GetChildren()) do
                     if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") then
                        if v.Name == "Jeremy [Lv. 850] [Boss]" then
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Size = Vector3.new(60,60,60)
                           v.Humanoid:ChangeState(8)
                           Hit = true
                           TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeedF)
                           repeat wait()
                              if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                                 game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 25, 0)
                              else
                                 Tweening = true
                                 TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeedF)
                                 Tweening = false
                              end
                           until v.Humanoid.Health <= 0 or BartiloQ == false
                           Hit = false
                           print("Kill Jeremy Success")
                           wait(5)
                           LocalPlayer.Character.Humanoid.Health = 0
                        end
                     end
                  end
                  wait(10)
                  for icount,v in pairs(game:GetService("Workspace").Map.Dressrosa.BartiloPlates:GetDescendants()) do
                     if v:IsA("TouchTransmitter") then

                        Plate1 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate1
                        Plate2 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate2
                        Plate3 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate3
                        Plate4 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate4
                        Plate5 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate5
                        Plate6 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate6
                        Plate7 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate7
                        Plate8 = game:GetService("Workspace").Map.Dressrosa.BartiloPlates.Plate8
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate1, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate1, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate2, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate2, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate3, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate3, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate4, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate4, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate5, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate5, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate6, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate6, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate7, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate7, 1) -- 1 is untouch
                              
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate8, 0) --0 is touch
                        wait()
                        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, Plate8, 1) -- 1 is untouch
                        wait(1)
                     end
                  end
                  wait(1)
                  LocalPlayer.Character.Humanoid.Health = 0
                  wait(3)
                  BartiloQ = false
                  wait(5)
                  if ChangeAF == true then
                     AF = trues
                     ChangeAF = false
                  end
                  if ChangeAFM == true then
                     AFM = true
                     ChangeAFM = false
                  end
      
               end
            end
         end
      end
      spawn(function()
         while Mode == "Magnet" and MagBartilo do
            game:GetService("RunService").RenderStepped:wait()
            BartiloQuest()
            pcall(function()
               repeat --testmagnet.BartiloQuest[1].OnComplete(Ms,PosMon)
                  game:GetService("RunService").RenderStepped:wait()
                  for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                     if v.Name == Mobx and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                        zeroGrav(v.HumanoidRootPart)
                        v.Humanoid.PlatformStand = false
                        v.HumanoidRootPart.Transparency = 1
                        v.HumanoidRootPart.CanCollide = false
                        v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                        v.HumanoidRootPart.CFrame = PosBartilo
                        v.Humanoid.WalkSpeed = 8
                     end
                  end
               until LocalPlayer.PlayerGui.Main.Quest.Visible == false or AFM == false or AF == false or MagnetOn == false
            end)
         end
      end)
   end)
end

if ThirdWorld then
   AutoFarm:addToggle("Auto Elite/Emma",getgenv().AutoElites,function(x)
      getgenv().AutoElites = x
   end)
end
spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do 
      if getgenv().AutoElites then
         wait(0.1)
         local args = {[1] = "EliteHunter"}
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
         for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
            if (v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and string.find(v.Name, "Diablo")) or
            (v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and string.find(v.Name, "Urban")) or
            (v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and string.find(v.Name, "Deandre")) then
               if AF == true then
                  AF = false
                  ChangeAF = true
               end
               if AFM == true then
                  AFM = false
                  ChangeAFM = true
               end
               local args = {[1] = "EliteHunter"}
               game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
               TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeed)
               repeat game:GetService("RunService").RenderStepped:wait(1)
                  for i,v in pairs(ListMelee) do
                     if game.Players.LocalPlayer.Backpack:FindFirstChild(ListMelee[i]) then
                        local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ListMelee[i])
                        game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
                     end
                  end
                  VirtualUser:CaptureController()
                  VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
                  VirtualUser:CaptureController()
                  VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
                  zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
                  zeroGrav(v.HumanoidRootPart)
                  v.Humanoid.PlatformStand = false
                  game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 15, 15)
                  v.HumanoidRootPart.Transparency = 0.9
                  v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                  v.HumanoidRootPart.CanCollide = false
                  v.HumanoidRootPart.Color = Color3.fromRGB(0, 0, 0)
               until v.Humanoid.Health <= 0 or getgenv().AutoElites == false
            elseif getgenv().AutoEliteHop == true and not game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == true then Teleport()
            end
         end
         if ChangeAF == true then
            AF = true
            ChangeAF = false
         end
         if ChangeAFM == true then
            AFM = true
            ChangeAFM = false
         end
         wait(2)
         if getgenv().AutoEliteHop == true then Teleport() end
      end
   end
end)
spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do  
         if v:IsA("Tool") then
            if v:FindFirstChild("RemoteFunctionShoot") then 
               SelectToolWeaponGun = v.Name
            end
         end
      end
   end
end)

local Boss = {}

for i, v in pairs(game.ReplicatedStorage:GetChildren()) do
	if string.find(v.Name, "Boss") then
		if v.Name == "Ice Admiral [Lv. 700] [Boss]" then
		else
			table.insert(Boss, v.Name)
		end
	end
end

for i, v in pairs(game.workspace.Enemies:GetChildren()) do
	if string.find(v.Name, "Boss") then
		if v.Name == "Ice Admiral [Lv. 700] [Boss]" then
		else
			table.insert(Boss, v.Name)
		end
	end
end

AutoFarm:addToggle("Auto Boss",false,function(Value)
	wait(.1)
	local args = {
		[1] = "AbandonQuest"
	}
	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
	FarmBoss = Value
   while FarmBoss do
      wait(0.1)
      AutoFarmBoss()
	end
end)

local AutoBossDD = AutoFarm:addDropdown("SelectBoss",Boss,function(Value)
   SelectBoss = Value
end)

AutoFarm:addToggle("AutoFarm All Boss",getgenv().AutoAllBoss, function(x)
   Allboss = x
   print(Allboss)
   while Allboss do
      wait(0.1)
      EquipWeapon(SelectToolWeapon)
      pcall(function()
         for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
            if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and v.Name ~= "rip_indra [Lv. 1500] [Boss]" and v.Name ~= "Ice Admiral [Lv. 700] [Boss]" and string.find(v.Name, "Boss") and Allboss then
               TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeed)
               Hit = true
               repeat wait(.1)
                  if Allboss and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 300 then
                     zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
                     zeroGrav(v.HumanoidRootPart)
                     game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 25, 0)
                     v.HumanoidRootPart.Transparency = 0.8
                     v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                     v.HumanoidRootPart.CanCollide = false
                     v.HumanoidRootPart.Color = Color3.fromRGB(0, 0, 0)
                  end
               until v.Humanoid.Health <= 0 or Allboss == false
               Hit = false
            end
         end
      end)
   end
end)

local function refreshBoss()

   table.clear(Boss)

   for i, v in pairs(game.ReplicatedStorage:GetChildren()) do
      if string.find(v.Name, "Boss") then
         if v.Name == "Ice Admiral [Lv. 700] [Boss]" then
         else
            table.insert(Boss, v.Name)
         end
      end
   end
   
   for i, v in pairs(game.workspace.Enemies:GetChildren()) do
      if string.find(v.Name, "Boss") then
         if v.Name == "Ice Admiral [Lv. 700] [Boss]" then
         else
            table.insert(Boss, v.Name)
         end
      end
   end
   

end


AutoFarm:addButton("Refresh Boss",function()
   AutoFarm:updateDropdown(AutoBossDD,"Refresh Boss",refreshBoss(),function()
   end)
end)



AutoFarm:addToggle("Mob Aura",false,function(x)
   MobAura = x
   while MobAura  do
      game:GetService("RunService").RenderStepped:wait()
      Hit = true
      pcall(function()
         for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
            if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
               MobAuraBringPos = v.HumanoidRootPart.CFrame
               repeat game:GetService("RunService").RenderStepped:wait()
                  EquipWeapon(SelectToolWeapon)
                  zeroGrav(v.HumanoidRootPart)
                  v.HumanoidRootPart.Transparency = 0.9
                  v.HumanoidRootPart.CanCollide = false
                  v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                  v.Humanoid.WalkSpeed = 1
                  game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 25, 0)
               until not v.Parent or v.Humanoid.Health <= 0 or MobAura == false
            end
         end
      end)
      Hit = false
   end
end)

Wapon = {}

for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do  
    if v:IsA("Tool") then
        table.insert(Wapon ,v.Name)
    end
end

for i,v in pairs(game.Players.LocalPlayer.Character:GetChildren()) do  
   if v:IsA("Tool") then
      table.insert(Wapon ,v.Name)
   end
end

local SelectWeapon = AutoFarm:addDropdown("Select Weapon",Wapon,function(Value)
    SelectToolWeapon = Value
    SelectToolWeaponNW = Value
    AutoTool = Value
end)
local function refreshWeapon1()

   table.clear(Wapon)

   for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do  
      if not table.find(Wapon, v.name) then
         if v:IsA("Tool") then
            table.insert(Wapon ,v.Name)
         end
      end
  end

   for i,v in pairs(game.Players.LocalPlayer.Character:GetChildren()) do  
      if not table.find(Wapon, v.name) then
         if v:IsA("Tool") then
            table.insert(Wapon ,v.Name)
         end
      end
   end

end

AutoFarm:addButton("Refresh Weapon",function()
   AutoFarm:updateDropdown(SelectWeapon,"Refresh Weapon",refreshWeapon1(),function()
   end)
end)



AutoFarm:addToggle("Auto Farm Observation",getgenv().Observation,function(a)
	getgenv().Observation = a
end)

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      if getgenv().Observation and game.Players.LocalPlayer.PlayerGui:FindFirstChild("ScreenGui") ~= nil then
         VirtualUser:CaptureController()
         VirtualUser:SetKeyDown('0x65')
         wait(2)
         VirtualUser:SetKeyUp('0x65')
      end
	while getgenv().Observation do wait()
		if SecondWorld then
			if game.Workspace.Enemies:FindFirstChild("Snow Lurker [Lv. 1375]") then
				if game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then
               TeleportTween(game.Workspace.Enemies:FindFirstChild("Snow Lurker [Lv. 1375]").HumanoidRootPart.CFrame,TweenSpeed)
					repeat wait()
						game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Snow Lurker [Lv. 1375]").HumanoidRootPart.CFrame * CFrame.new(3,0,0)
					until getgenv().Observation == false or not game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel")
				else
               TeleportTween(game.Workspace.Enemies:FindFirstChild("Snow Lurker [Lv. 1375]").HumanoidRootPart.CFrame,TweenSpeed)

					repeat wait()
						game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Snow Lurker [Lv. 1375]").HumanoidRootPart.CFrame * CFrame.new(20,25,0)
                  if not  game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then

                     VirtualUser:CaptureController()
                     VirtualUser:SetKeyDown('0x65')
                     VirtualUser:SetKeyUp('0x65')
                     end
					until getgenv().Observation == false or game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel")
				end
			else
				TeleportTween(CFrame.new(5496.42236, 28.8430767, -6766.56787),TweenSpeed)
			end
		elseif FirstWorld then
			if game.Workspace.Enemies:FindFirstChild("Galley Captain [Lv. 650]") then
				if game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then
               TeleportTween(game.Workspace.Enemies:FindFirstChild("Galley Captain [Lv. 650]").HumanoidRootPart.CFrame,TweenSpeed)
					repeat wait()
						game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Galley Captain [Lv. 650]").HumanoidRootPart.CFrame * CFrame.new(3,0,0)
					until getgenv().Observation == false or not game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel")
				else
               TeleportTween(game.Workspace.Enemies:FindFirstChild("Galley Captain [Lv. 650]").HumanoidRootPart.CFrame,TweenSpeed)

					repeat wait()
						game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Galley Captain [Lv. 650]").HumanoidRootPart.CFrame * CFrame.new(20,25,0)
                  if not  game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then
                     game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Galley Captain [Lv. 650]").HumanoidRootPart.CFrame * CFrame.new(20,25,0)

                     VirtualUser:CaptureController()
                     VirtualUser:SetKeyDown('0x65')
                     VirtualUser:SetKeyUp('0x65')
                     end
					until getgenv().Observation == false or game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel")
				end
			else
            TeleportTween(CFrame.new(5533.29785, 88.1079102, 4852.3916,TweenSpeed))
         end
         elseif ThirdWorld then
            if game.Workspace.Enemies:FindFirstChild("Marine Commodore [Lv. 1700]") then
               if game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then
                  TeleportTween(game.Workspace.Enemies:FindFirstChild("Marine Commodore [Lv. 1700]").HumanoidRootPart.CFrame,TweenSpeed)
                  repeat wait()
                     game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Marine Commodore [Lv. 1700]").HumanoidRootPart.CFrame * CFrame.new(3,0,0)
                  until getgenv().Observation == false or not game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel")
               else
                  TeleportTween(game.Workspace.Enemies:FindFirstChild("Marine Commodore [Lv. 1700]").HumanoidRootPart.CFrame,TweenSpeed)
                  repeat wait()
                     game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace.Enemies:FindFirstChild("Marine Commodore [Lv. 1700]").HumanoidRootPart.CFrame * CFrame.new(20,25,0)
                     if not  game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then
                     VirtualUser:CaptureController()
                     VirtualUser:SetKeyDown('0x65')
                     VirtualUser:SetKeyUp('0x65')
                     end
                  until getgenv().Observation == false or game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel")
               end
            else
               TeleportTween(CFrame.new(5533.29785, 88.1079102, 4852.3916),TweenSpeed)
			   end
		   end
	   end
   end
end)

AutoFarm:addSlider("Cooldown Ken(No use for now)", 0,0,300,function(value)
	Cooldownken = value
end)
--Legendary Sword--
if SecondWorld then
   AutoFarm:addToggle("AutoBuy LegendarySword",getgenv().LegendarySword,function(Value)
      if Value == false then
         LegendarySword = Value    
         getgenv().LegendarySword = Value
         
      else LegendarySword = Value AutoLegendarySword()
      end
   end)
end

function AutoLegendarySword()
   while LegendarySword and SecondWorld do
      wait()
      if MyBeli>= 2000000 then
         local args = {[1] = "LegendarySwordDealer",[2] = "2"}
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end
   end
end
--Enhancement
if SecondWorld or ThirdWorld then
   AutoFarm:addToggle("Auto Buy Enhancement (Use 1500 Fragments)",false,function(Value) 
      Enhancement = Value   
      AutoEnhancement()
   end)
end

function AutoEnhancement()
	while Enhancement and (SecondWorld or ThirdWorld) do
      wait()
      if MyFragment >= 1500 then
         local args = {[1] = "ColorsDealer",[2] = "2"}
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end 
	end
end


SkillZ = true
SkillX = true
SkillC = true
SkillV = true
SkillF = true
SkillZTime = 0.5 
SkillXTime = 0.5 
SkillCTime = 0.5
SkillVTime = 0.5
SkillFTime = 0.5
game:GetService("RunService").Heartbeat:Connect(
   function()
      if _G.FruitMastery then
         zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
      end
      if _G.GunMastery then
         zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
     end
   end
)  

WeaponMastery = ""
AutoFarm:addToggle("Auto Farm Mastery Fruit",false,function(v)
   if WeaponMastery == "" and v then
   else
      CheckQuest()
      _G.FruitMastery = v
      Ms = ""
   end	
   while _G.FruitMastery do wait()
      AutoDevilFruitMastery()
   end
end)
LocalPlayer = game.Players.LocalPlayer
function AutoDevilFruitMastery()
   if _G.FruitMastery then
      if LocalPlayer.PlayerGui.Main.Quest.Visible == false then
         StatrMagnetDevilFruitMastery = false
         CheckQuest()
         TeleportTween(CFrameQuest,TweenSpeed)
         wait(1.1)
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuest, LevelQuest)
      elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible == true then
         CheckQuest()
         TeleportTween(CFrameMon,TweenSpeed)
         if game:GetService("Workspace").Enemies:FindFirstChild(Ms) then
            pcall(function()
               for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                  CheckQuest()  
                  if v.Name == Ms then
                     repeat wait() CheckQuest()  
                        if game:GetService("Workspace").Enemies:FindFirstChild(Ms) then
                           if string.find(LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text, NameMon) then
                              if not game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
                                 local args = { [1] = "Buso"}
                                 game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                              end
                              HealthMin = v.Humanoid.MaxHealth*Persen/100
                              PosMon = v.HumanoidRootPart.CFrame
                           if v.Humanoid.Health <= HealthMin then
                              UseDF = true
                              EquipWeapon(game.Players.LocalPlayer.Data.DevilFruit.Value)
                              v.HumanoidRootPart.CanCollide = false
                              v.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
                              v.HumanoidRootPart.Transparency = 0.75
                              TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,0,10),TweenSpeed)
                              if game:GetService("Players").LocalPlayer.Character:FindFirstChild(game.Players.LocalPlayer.Data.DevilFruit.Value) then

                                 if SkillZ and v.Humanoid.Health <= HealthMin then
                                    local args = {
                                    [1] = v.HumanoidRootPart.Position
                                    }
                                    game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Data.DevilFruit.Value].RemoteEvent:FireServer(unpack(args))
                                    local N=game:GetService("VirtualInputManager")
                                    N:SendKeyEvent(true,"Z",false,game)
                                    wait(SkillZTime)
                                    N:SendKeyEvent(false,"Z",false,game)
                                 end

                                 if SkillX and v.Humanoid.Health <= HealthMin then
                                    local args = {
                                    [1] = v.HumanoidRootPart.Position
                                    }
                                    game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Data.DevilFruit.Value].RemoteEvent:FireServer(unpack(args))
                                    local N=game:GetService("VirtualInputManager")
                                    N:SendKeyEvent(true,"X",false,game)
                                    wait(SkillXTime)
                                    N:SendKeyEvent(false,"X",false,game)
                                 end
                                 
                                 if SkillC and v.Humanoid.Health <= HealthMin then
                                    local args = {
                                       [1] = v.HumanoidRootPart.Position
                                    }
                                    game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Data.DevilFruit.Value].RemoteEvent:FireServer(unpack(args))
                                    local N=game:GetService("VirtualInputManager")
                                    N:SendKeyEvent(true,"C",false,game)
                                    wait(SkillCTime)
                                    N:SendKeyEvent(false,"C",false,game)
                                 end

                                 if SkillV and v.Humanoid.Health <= HealthMin then
                                    local args = {
                                       [1] = v.HumanoidRootPart.Position
                                    }
                                    game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Data.DevilFruit.Value].RemoteEvent:FireServer(unpack(args))
                                    local N=game:GetService("VirtualInputManager")
                                    N:SendKeyEvent(true,"V",false,game)
                                    wait(SkillVTime)
                                    N:SendKeyEvent(false,"V",false,game)
                                 end
                                 if SkillF and v.Humanoid.Health <= HealthMin then
                                    local args = {
                                       [1] = v.HumanoidRootPart.Position
                                    }
                                    game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Data.DevilFruit.Value].RemoteEvent:FireServer(unpack(args))
                                    local N=game:GetService("VirtualInputManager")
                                    N:SendKeyEvent(true,"F",false,game)
                                    wait(SkillFTime)
                                    N:SendKeyEvent(false,"F",false,game)
                                 end
                              end   
                           else
                              UseDF = false
                              EquipWeapon(WeaponMastery)
                              if HideHitBlox then
                                 v.HumanoidRootPart.Transparency = 1
                              else
                                 v.HumanoidRootPart.Transparency = 0.75
                              end
                              v.HumanoidRootPart.CanCollide = false
                              v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                              v.Humanoid.PlatformStand = true
                              TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,15,0),TweenSpeed)
                              game:GetService'VirtualUser':CaptureController()
                              game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))  
                           end
                           StatrMagnetDevilFruitMastery = true 
                        else
                           StatrMagnet = false
                           CheckQuest()
                  
                           TeleportTween(CFrameQuest,TweenSpeed)
                           wait(1.5)
                           game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuest, LevelQuest)
                        end  
                     else
                        CheckQuest() 
                        TeleportTween(CFrameMon,TweenSpeed)
                     end 
                     until not v.Parent or v.Humanoid.Health <= 0 or _G.AutoFarm == false or LocalPlayer.PlayerGui.Main.Quest.Visible == false
                     StatrMagnetDevilFruitMastery = false
                     CheckQuest() 
                     TeleportTween(CFrameMon,TweenSpeed)
                  end
               end
            end)   
         else
         CheckQuest()
         TeleportTween(CFrameMon,TweenSpeed)
         end 
      end
   end
end

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do  
         if v:IsA("Tool") then
            if v:FindFirstChild("RemoteFunctionShoot") then 
               SelectToolWeaponGun = v.Name
            end
         end
      end
   end
end)

AutoFarm:addToggle("Auto Farm Gun Mastery",false,function(v)
   if WeaponMastery == "" and v then
   else 
      CheckQuest()

      _G.GunMastery = v
      Ms = ""
   end	
   while _G.GunMastery do wait()
      AutoGunMastery()
   end
end)
function AutoGunMastery()
   if game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == false then  
      CheckQuest()
      TeleportTween(CFrameQuest,TweenSpeed)
      wait(1.1)
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuest, LevelQuest)
      wait(0.5)
      TeleportTween(CFrameMon,TweenSpeed)
   elseif game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == true then  
      for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
         CheckQuest()
         pcall(function()
            if game.Workspace.Enemies:FindFirstChild(Ms) then
               if _G.GunMastery and v.Name == Ms then
                  if setsimulationradius then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  repeat wait()
                     pcall(function()
                        if game.Workspace.Enemies:FindFirstChild(Ms) then
                           if string.find(LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text, NameMon) then
                              HealthMin = v.Humanoid.MaxHealth*Persen/100
                              PosMon = v.HumanoidRootPart.CFrame
                              if v.Humanoid.Health <= HealthMin then
                                 EquipWeapon(SelectToolWeaponGun)
                                 v.HumanoidRootPart.CanCollide = false
                                 v.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
                                 v.HumanoidRootPart.Transparency = 0.75
                                 TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,0,10),TweenSpeed)
                                 local args = {
                                    [1] = v.HumanoidRootPart.Position,
                                    [2] = v.HumanoidRootPart
                                 }
                                 game:GetService("Players").LocalPlayer.Character[SelectToolWeaponGun].RemoteFunctionShoot:InvokeServer(unpack(args))
                              else

                                 EquipWeapon(WeaponMastery)

                                 if HideHitBlox then
                                    v.HumanoidRootPart.Transparency = 1
                                 else
                                    v.HumanoidRootPart.Transparency = 0.75
                                 end

                                 v.HumanoidRootPart.CanCollide = false
                                 v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                                 v.Humanoid.PlatformStand = true
                                 TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0,15,0),TweenSpeed)
                                 game:GetService'VirtualUser':CaptureController()
                                 game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                              end 
                              StatrMagnet = true
                           else
                              CheckQuest()
                              
                              TeleportTween(CFrameQuest,TweenSpeed)
                              wait(1.1)

                              wait(0.5)
                              TeleportTween(CFrameMon,TweenSpeed)
                           end
                        else
                           CheckQuest()
                           TeleportTween(CFrameMon,TweenSpeed)
                        end
                     end)
                  until game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == false or _G.GunMastery == false or v.Humanoid.Health <= 0 or not v.Parent or v.Humanoid.Health <= 0
               end
            else
               CheckQuest()
               TeleportTween(CFrameMon,TweenSpeed)
            end
         end)
      end
   end
end

Persen = 15

AutoFarm:addSlider("Health %", Persen, 1,100,function(v)
   Persen = v
end)

local AMS = AutoFarm:addDropdown("Select Weapon",Wapon,function(v)
   WeaponMastery = v
   WeaponOldMastery = v
end)
   local function refreshWeapon2()

      table.clear(Wapon)

      for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do  
         if not table.find(Wapon, v.name) then
            if v:IsA("Tool") then
               table.insert(Wapon ,v.Name)
            end
         end
      end

      for i,v in pairs(game.Players.LocalPlayer.Character:GetChildren()) do  
         if not table.find(Wapon, v.name) then
            if v:IsA("Tool") then
               table.insert(Wapon ,v.Name)
            end
         end
      end

   end


AutoFarm:addButton("Refresh Weapon",function()
   AutoFarm:updateDropdown(AMS,"Refresh Weapon2",refreshWeapon2(),function()
   end)
end)

AutoFarm:addToggle("Auto Equip", "", false,function(Value)
   if SelectToolWeapon == "" then
   else
	   autoequip = Value
   end
end)


spawn(function() 
   while game:GetService("RunService").RenderStepped:wait() do
      if autoequip then
         for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
            if v.Name == AutoTool then 
               game.Players.LocalPlayer.Character.Humanoid:EquipTool(v)
            end 
         end
      end 
   end
end)

spawn(function()  -- Noclip
   while game:GetService("RunService").RenderStepped:wait() do
      if AF == true or BartiloQuest == true or AutoSecondWorld == true or AutoThirdworld == true or AFM == true or Allboss == true or MobAura == true then
         pcall(function ()
            zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
           end)
      end 
   end
end)


AutoFarm:addToggle("Auto BusoHaki",getgenv().AutoBuso,function(Value)
   getgenv().AutoBuso = Value
end)

AutoFarm:addToggle("Auto KenHaki",getgenv().AutoKen,function(Value)
   getgenv().AutoKen = Value  
end)

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      if getgenv().AutoBuso then 
         if game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
         else
            local args = {[1] = "Buso"}
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
         end
      end
      if getgenv().AutoKen then 
         if game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("ImageLabel") then

         else wait(1)
            local virtualUser = game:GetService('VirtualUser')
            virtualUser:CaptureController()

            virtualUser:SetKeyDown('0x65')
            wait(2)
            virtualUser:SetKeyUp('0x65')
         end
      end
   end
end)

wait()

local StatsPlr = lib:addPage("Stats", 7061402283 )
local StatsLPlr = StatsPlr:addSection("Auto Stats")

melee = false
StatsLPlr:addToggle("Melee",false,function(Value)
    melee = Value  
   while melee  do
      wait()
      if game.Players.localPlayer.Data.Points.Value >= PointStats then
         local args = {
            [1] = "AddPoint",
            [2] = "Melee",
            [3] = PointStats
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end 
   end
end)

defense = false
StatsLPlr:addToggle("Defense",false,function(Value)
    defense = Value
    while defense  do
      wait()
      if game.Players.localPlayer.Data.Points.Value >= PointStats then
         local args = {
            [1] = "AddPoint",
            [2] = "Defense",
            [3] = PointStats
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end 
   end
end)

sword = false
StatsLPlr:addToggle("Sword",false,function(Value)
    sword = Value
    while sword  do
      wait()
      if game.Players.localPlayer.Data.Points.Value >= PointStats then
         local args = {
            [1] = "AddPoint",
            [2] = "Sword",
            [3] = PointStats
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end 
   end
end)

gun = false
StatsLPlr:addToggle("Gun",false,function(Value)
    gun = Value
    while gun do
      wait()
      if game.Players.localPlayer.Data.Points.Value >= PointStats then
         local args = {
            [1] = "AddPoint",
            [2] = "Gun",
            [3] = PointStats
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end 
   end
end)

demonfruit = false
StatsLPlr:addToggle("Devil Fruit",false,function(Value)
    demonfruit = Value
    while demonfruit  do
      wait()
      if game.Players.localPlayer.Data.Points.Value >= PointStats then
         local args = {
            [1] = "AddPoint",
            [2] = "Demon Fruit",
            [3] = PointStats
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end 
   end
end)

PointStats = 1
StatsLPlr:addSlider("Point ",1,1,10,function(a)
    PointStats = a
end)

local StatsPlr2 = StatsPlr:addSection("Players Stats")

StatsPlr2:addButton("Check All Stats",function()
   local Level = tostring(game:GetService("Players")["LocalPlayer"].Data.Level.Value)
   lib:Notify("Level:",Level)
   wait(2)
   local Beli = tostring(game:GetService("Players")["LocalPlayer"].Data.Beli.Value)
   lib:Notify("Beli:",Beli)
   wait(2)
   local F = tostring(game:GetService("Players")["LocalPlayer"].Data.Fragments.Value)
   lib:Notify("Fragments:",F)
   wait(2)
   local Race = tostring(game:GetService("Players")["LocalPlayer"].Data.Race.Value)
   lib:Notify("Race:",Race)
   wait(2)
   local DF = tostring(game:GetService("Players")["LocalPlayer"].Data.DevilFruit.Value)
   lib:Notify("Devil Fruit:",DF)
   wait(2)
   local s = tostring(game:GetService("Players")["LocalPlayer"].Data.SpawnPoint.Value)
   lib:Notify("Check SpawnPoint:",s)
   wait(2)
   local BH = tostring(game:GetService("Players")["LocalPlayer"].leaderstats["Bounty/Honor"].Value)
   lib:Notify("Bounty/Honor:",BH)
end)

StatsPlr2:addButton("Check Ectoplasm",function()
   lib:Notify("You Have:",game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Ectoplasm","Check").."-Ectoplasm")
end)

StatsPlr2:addButton("Check KenHaki", function() 
   lib:Notify("You Have:",game:GetService("Players")["LocalPlayer"].VisionRadius.Value.."-Exp")
end)

wait()



function tp(x,y,z)
   -- SETTINGS START
   local valtomove = 6.5
   noclip = true
   anchored = false
   -- SETTINGS END
   
   
   moving = true
   if x < game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X then
   while x < game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X and game.Players.LocalPlayer.Character.Humanoid.Health > 0 do
   game:GetService("RunService").RenderStepped:wait() 
   game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Vector3.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X-valtomove,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z))
   end
   end
   if z < game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z then
   while z < game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z and game.Players.LocalPlayer.Character.Humanoid.Health > 0 do
   game:GetService("RunService").RenderStepped:wait()
   game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Vector3.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z-valtomove))
   end
   end
   if x > game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X then
   while x > game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X and game.Players.LocalPlayer.Character.Humanoid.Health > 0 do
   game:GetService("RunService").RenderStepped:wait()
   game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Vector3.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X+valtomove,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z))
   end
   end
   if z > game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z then
   while z > game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z and game.Players.LocalPlayer.Character.Humanoid.Health > 0 do
   game:GetService("RunService").RenderStepped:wait()
   game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Vector3.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z+valtomove))
   end
   end
   if y < game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y then
   while y < game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y and game.Players.LocalPlayer.Character.Humanoid.Health > 0 do
   game:GetService("RunService").RenderStepped:wait()
   game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Vector3.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y-valtomove,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z))
   end
   end
   if y > game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y then
   while y > game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y and game.Players.LocalPlayer.Character.Humanoid.Health > 0 do
   game:GetService("RunService").RenderStepped:wait()
   game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(Vector3.new(game.Players.LocalPlayer.Character.HumanoidRootPart.Position.X,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y+valtomove,game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Z))
   end
   end
   moving = false
   game.Players.LocalPlayer.Character:MoveTo(Vector3.new(x,y,z))
   if anchored == true then
   game.Players.LocalPlayer.Character.Head.Anchored = false
   end
   end
   
   spawn (function()
   game:getService("RunService"):BindToRenderStep("",0,function()
   if not game.Players.LocalPlayer.Character:findFirstChildOfClass("Humanoid") then return end
   if moving == true then
   if noclip == true then
      zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
   end
   if anchored == true then
   game.Players.LocalPlayer.Character.Head.Anchored = true
   game.Players.LocalPlayer.Character.HumanoidRootPart = true
   end
   end
   end)
   end)
   
local Teleport = lib:addPage("Teleport", 7061398829)
local TeleportFunction = Teleport:addSection("Teleport Function")

TeleportFunction:addButton("Stop Tween", function()
   TeleportTween(game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame,TweenSpeed)
end)


TeleportFunction:addToggle("Ctrl + Click = TP",false,function(vu)
   CTRL = vu
end)

local Plr = game:GetService("Players").LocalPlayer
local Mouse = Plr:GetMouse()
Mouse.Button1Down:connect(
   function()
      if not game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.LeftControl) then
         return
      end
      if not Mouse.Target then
         return
      end
      if CTRL then
         Plr.Character:MoveTo(Mouse.Hit.p)
      end
   end
)

Flower = {
   "Blue Flower";
   "Red Flower";
}

TeleportFunction:addDropdown("Teleport Flower",Flower,function(v)

   FlowerSelected = v
   if FlowerSelected == "Red Flower" then
      for i,v in pairs(game.Workspace:GetDescendants()) do
         if v.Name == "Flower2" then
            TeleportTween(v.CFrame,TweenSpeed)
         end
      end
   end

   if FlowerSelected == "Blue Flower" then
      for i,v in pairs(game.Workspace:GetDescendants()) do
         if v.Name == "Flower1" then
            TeleportTween(v.CFrame,TweenSpeed)
         end
      end
   end
end)

TeleportFunction:addButton("Teleport to SeaBeasts", function()
   for i,v in pairs(game.Workspace.SeaBeasts:GetChildren()) do
      if v:FindFirstChild("HumanoidRootPart") then
         SeaBeast = v.HumanoidRootPart.CFrame * CFrame.new(0,100,0)
         TeleportTween(SeaBeast,TweenSpeed)
      end
   end
end)

local TeleportPlace = Teleport:addSection("Teleport Place")
if SecondWorld then
   

	TeleportPlace:addButton("First Spot", function()
		TeleportTween(CFrame.new(82.9490662, 18.0710983, 2834.98779),TweenSpeed)
	end)
	TeleportPlace:addButton("Cafe", function()
		TeleportTween(CFrame.new(-385.250916, 73.0458984, 297.388397),TweenSpeed)
	end)

	TeleportPlace:addButton("Kingdom of Rose", function()
		TeleportTween(game.Workspace["_WorldOrigin"].Locations["Kingdom of Rose"].CFrame,TweenSpeed)
	end)
	TeleportPlace:addButton("Factory", function()
		TeleportTween(CFrame.new(430.42569, 210.019623, -432.504791),TweenSpeed)
	end)

	TeleportPlace:addButton("Doflamingo Mansion", function()
		TeleportTween(CFrame.new(-390.096313, 331.886475, 673.464966),TweenSpeed)
	end)
	TeleportPlace:addButton("Doflamingo Room", function()
		TeleportTween(CFrame.new(2302.19019, 15.1778421, 663.811035),TweenSpeed)
	end)
   
	TeleportPlace:addButton("Dark Arena", function()
		TeleportTween(game.Workspace["_WorldOrigin"].Locations["Dark Arena"].CFrame,TweenSpeed)
	end)
	TeleportPlace:addButton("Green bit", function()
		TeleportTween(CFrame.new(-2372.14697, 72.9919434, -3166.51416),TweenSpeed)
	end)

	TeleportPlace:addButton("Colosseum", function()
		TeleportTween(CFrame.new(-1836.58191, 44.5890656, 1360.30652),TweenSpeed)
	end)
	TeleportPlace:addButton("Ghost Island", function()
		TeleportTween(CFrame.new(-5571.84424, 195.182297, -795.432922),TweenSpeed)
	end)

	TeleportPlace:addButton("Ghost Island 2nd", function()
		TeleportTween(CFrame.new(-5931.77979, 5.19706631, -1189.6908),TweenSpeed)
	end)
	TeleportPlace:addButton("Snow Mountain", function()
		TeleportTween(CFrame.new(1384.68298, 453.569031, -4990.09766),TweenSpeed)
	end)

	TeleportPlace:addButton("Cold Island", function()
		TeleportTween(CFrame.new(-6026.96484, 14.7461271, -5071.96338),TweenSpeed)
	end)
	TeleportPlace:addButton("Magma Island", function()
		TeleportTween(CFrame.new(-5478.39209, 15.9775667, -5246.9126),TweenSpeed)
	end)

	TeleportPlace:addButton("Cursed Ship", function()
		TeleportTween(CFrame.new(902.059143, 124.752518, 33071.8125),TweenSpeed)
	end)
	TeleportPlace:addButton("Ice Castle", function()
		TeleportTween(CFrame.new(5400.40381, 28.21698, -6236.99219),TweenSpeed)
	end)

	TeleportPlace:addButton("Forgotten Island", function()
		TeleportTween(CFrame.new(-3043.31543, 238.881271, -10191.5791),TweenSpeed)
	end)
	TeleportPlace:addButton("Usoapp Island", function()
		TeleportTween(CFrame.new(4748.78857, 8.35370827, 2849.57959),TweenSpeed)
	end)

	TeleportPlace:addButton("Minisky Island", function()
		TeleportTween(CFrame.new(-260.358917, 49325.7031, -35259.3008),TweenSpeed)
	end)
   TeleportPlace:addButton("Indra Island", function()
		TeleportTween(CFrame.new(-26697.9883, 8.5493145, 425.623444, 0.15644598, 0, 0.987686574, 0, 1, 0, -0.987686574, 0, 0.15644598),TweenSpeed)
	end)

elseif FirstWorld then

	TeleportPlace:addButton("Start Island", function()
		TeleportTween(CFrame.new(1071.2832, 16.3085976, 1426.86792),TweenSpeed)
	end)
	TeleportPlace:addButton("Marine Start", function()
		TeleportTween(CFrame.new(-2573.3374, 6.88881969, 2046.99817),TweenSpeed)
	end)
	TeleportPlace:addButton("Middle Town", function()
		TeleportTween(CFrame.new(-655.824158, 7.88708115, 1436.67908),TweenSpeed)
	end)

	TeleportPlace:addButton("Jungle", function()
		TeleportTween(CFrame.new(-1249.77222, 11.8870859, 341.356476),TweenSpeed)
	end)
	TeleportPlace:addButton("Pirate Village", function()
		TeleportTween(CFrame.new(-1122.34998, 4.78708982, 3855.91992),TweenSpeed)
	end)

	TeleportPlace:addButton("Desert", function()
		TeleportTween(CFrame.new(1094.14587, 6.47350502, 4192.88721),TweenSpeed)
	end)
	TeleportPlace:addButton("Frozen Village", function()
		TeleportTween(CFrame.new(1198.00928, 27.0074959, -1211.73376),TweenSpeed)
	end)

	TeleportPlace:addButton("MarineFord", function()
		TeleportTween(CFrame.new(-4505.375, 20.687294, 4260.55908),TweenSpeed)
	end)
	TeleportPlace:addButton("Colosseum", function()
		TeleportTween(CFrame.new(-1428.35474, 7.38933945, -3014.37305),TweenSpeed)
	end)

	TeleportPlace:addButton("Sky 1st Floor", function()
		TeleportTween(CFrame.new(-4970.21875, 717.707275, -2622.35449),TweenSpeed)
	end)
	TeleportPlace:addButton("Sky 2st Floor", function()
		TeleportTween(CFrame.new(-4813.0249, 903.708557, -1912.69055),TweenSpeed)
	end)

	TeleportPlace:addButton("Sky 3st Floor", function()
		TeleportTween(CFrame.new(-7952.31006, 5545.52832, -320.704956),TweenSpeed)
	end)
	TeleportPlace:addButton("Prison", function()
		TeleportTween(CFrame.new(4874.8125, 5.6519904136658, 735.57012939453),TweenSpeed)
	end)

	TeleportPlace:addButton("Magma Village", function()
		TeleportTween(CFrame.new(-5231.75879, 8.61593437, 8467.87695),TweenSpeed)
	end)
	TeleportPlace:addButton("UnderWater City", function()
		TeleportTween(CFrame.new(61163.8516, 11.7796879, 1819.78418),TweenSpeed)
	end)

	TeleportPlace:addButton("Fountain City", function()
		TeleportTween(CFrame.new(5132.7124, 4.53632832, 4037.8562),TweenSpeed)
	end)
	TeleportPlace:addButton("House Cyborg's", function()
		TeleportTween(CFrame.new(6262.72559, 71.3003616, 3998.23047),TweenSpeed)
	end)

	TeleportPlace:addButton("Shank's Room", function()
		TeleportTween(CFrame.new(-1442.16553, 29.8788261, -28.3547478),TweenSpeed)
	end)
	TeleportPlace:addButton("Mob Island", function()
		TeleportTween(CFrame.new(-2850.20068, 7.39224768, 5354.99268),TweenSpeed)
	end)

   TeleportPlace:addButton("NPC Teleport", function()
      Lib:Notify("Npc Teleport")
	end)

	TeleportPlace:addButton("Sword Dealer", function()
		TeleportTween(CFrame.new(895.941956, 14.9845877, 1412.76782, -0.939700961, 0, -0.341998369, 0, 1, 0, 0.341998369, 0, -0.939700961),TweenSpeed)
	end)
	TeleportPlace:addButton("Weapon Dealer", function()
		TeleportTween(CFrame.new(-700.12005615234, 7.8522434234619, 1512.2841796875),TweenSpeed)
	end)

	TeleportPlace:addButton("NPC Random Fruit", function()
		TeleportTween(CFrame.new(-1441.53357, 61.1999283, 7.7518611, 0.499959469, -0, -0.866048813, 0, 1, -0, 0.866048813, 0, 0.499959469),TweenSpeed)
	end)
	TeleportPlace:addButton("Sword Dealer of the West", function()
		TeleportTween(CFrame.new(-1279.0747070313, 13.752041816711, 3990.2504882813),TweenSpeed)
	end)

	TeleportPlace:addButton("Darkleg Teacher", function()
		TeleportTween(CFrame.new(-983.618103, 12.450016, 3990.46265, 0.358573437, 0, 0.933501542, 0, 1, 0, -0.933501542, 0, 0.358573437),TweenSpeed)
	end)
	TeleportPlace:addButton("Hasan", function()
		TeleportTween(CFrame.new(1322.017578125, 15.891534805298, 4486.0834960938),TweenSpeed)
	end)

	TeleportPlace:addButton("Ability Teacher", function()
		TeleportTween(CFrame.new(1488.6115722656, 37.829216003418, -1412.5043945313),TweenSpeed)
	end)
	TeleportPlace:addButton("Sick man", function()
		TeleportTween(CFrame.new(1460.7014160156, 88.573440551758, -1392.4689941406),TweenSpeed)
	end)

	TeleportPlace:addButton("Sword dealer of the east", function()
		TeleportTween(CFrame.new(1431.3474121094, 87.752792358398, -1388.0727539063),TweenSpeed)
	end)
	TeleportPlace:addButton("Advance Weapon Dealer", function()
		TeleportTween(CFrame.new(-4997.841796875, 41.252048492432, 4403.181640625),TweenSpeed)
	end)

	TeleportPlace:addButton("Parlus", function()
		TeleportTween(CFrame.new(-4929.2255859375, 96.839538574219, 3868.7917480469),TweenSpeed)
	end)
	TeleportPlace:addButton("Master Sword Dealer", function()
		TeleportTween(CFrame.new(-4750.6059570313, 717.65997314453, -2654.2487792969),TweenSpeed)
	end)

	TeleportPlace:addButton("Usoap", function()
		TeleportTween(CFrame.new(-8036.05859375, 5756.033203125, -1934.3215332031),TweenSpeed)
	end)
	TeleportPlace:addButton("Remove Fruit", function()
		TeleportTween(CFrame.new(5665.552734375, 64.651931762695, 868.56658935547),TweenSpeed)
	end)

	TeleportPlace:addButton("SecondWorld Teleporter", function()
		TeleportTween(CFrame.new(-1164.2852783203, 8.2123336791992, 1727.9249267578),TweenSpeed)
	end)
	TeleportPlace:addButton("Kungfu Teacher", function()
		TeleportTween(CFrame.new(61581.54296875, 18.870784759521, 985.88067626953),TweenSpeed)
	end)

elseif ThirdWorld then

   TeleportPlace:addButton("Port Town", function()
      TeleportTween(CFrame.new(-279.197723, 16.72994041, 5367.32324, -0.91604501, -2.56747379e-09, -0.401075482, 9.37653066e-10, 1, -8.54304538e-09, 0.401075482, -8.20188362e-09, -0.91604501),TweenSpeed)
	end)
   TeleportPlace:addButton("Great Tree", function()
      TeleportTween(CFrame.new(2159.84717, 70.2814083, -6563.24609, 0.874622703, -0, -0.484804183, 0, 1, -0, 0.484804183, 0, 0.874622703),TweenSpeed)
	end)

   TeleportPlace:addButton("Hydra Island", function()
      TeleportTween(CFrame.new(5299.00098, 630.816284, -48.6855469, -0.707134247, 0, -0.707079291, 0, 1, 0, 0.707079291, 0, -0.707134247),TweenSpeed)
	end)
   TeleportPlace:addButton("Castle on the sea", function()
		TeleportTween(CFrame.new(-5026.3584, 323.515503, -2996.28442, -0.391219467, 5.66034792e-08, 0.920297444, 2.15400586e-08, 1, -5.23489341e-08, -0.920297444, -6.5666228e-10, -0.391219467),TweenSpeed)
	end)

   TeleportPlace:addButton("Pineapple Town", function()
      TeleportTween(CFrame.new(-11363.5166, 362.381439, -10327.9727, -0.258864403, 0, -0.965913713, 0, 1, 0, 0.965913713, 0, -0.258864403),TweenSpeed)
	end)
   TeleportPlace:addButton("Mansion", function()
      TeleportTween(CFrame.new(-12555.1846, 507.168274, -7480.31543),TweenSpeed)
	end)

   TeleportPlace:addButton("Floating Turtle", function()
		TeleportTween(CFrame.new(-12001.6152, 1707.39319, -8789.03711),TweenSpeed)
	end)
   TeleportPlace:addButton("Beautiful Pirate Domain", function()
		TeleportTween(CFrame.new(5310.80957, 160.446838, 129.390533, 1, 0, 0, 0, 1, 0, 0, 0, 1),TweenSpeed)
	end)
   TeleportPlace:addButton("Friendly Arena", function()
		TeleportTween(CFrame.new(5220.28955, 72.8193436, -1500.86304, 1, 0, 0, 0, 1, 0, 0, 0, 1),TweenSpeed)
	end)
   TeleportPlace:addButton("Soul Reaper Raid Room", function()
		TeleportTween(CFrame.new(-9522.0957, 315.89975, 6751.88818),TweenSpeed)
	end)
   TeleportPlace:addButton("Haunted Castle", function()
		TeleportTween(CFrame.new(-9530.61035, 200.860657, 5763.13477),TweenSpeed)
	end)
end
wait()

local Players = lib:addPage("Players", 7251993295)
local Pvp = Players:addSection("PvP")

PlayerName = {}

for i,v in pairs(game.Players:GetChildren()) do  
   table.insert(PlayerName ,v.Name)
end

local Player = Pvp:addDropdown("Selected Player",PlayerName,function(plys)
   KillPlayer = plys
end)

local function refreshPlayers()
   table.clear(PlayerName)
   for i,v in pairs(game.Players:GetChildren()) do
       if not table.find(PlayerName, v.Name) then
           table.insert(PlayerName, v.Name)
      
       end
   end
end

Pvp:addButton("Refresh Player",function()
   Pvp:updateDropdown(Player,"Refresh Player",refreshPlayers(),function(plys)
   end)
end)

Pvp:addToggle("Kill Player (Combat)",false,function(bool)

   pk = bool
   local plr1 = game.Players.LocalPlayer.Character
   local plr2 = game.Players:FindFirstChild(KillPlayer)

   repeat wait()

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 1)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 1)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 1)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 1)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 1)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -0)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0,-0)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -0)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -0)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -0)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -0)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -10)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 10)
      end)
      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -10)
      end)

      plr2.Character.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
      game:GetService'VirtualUser':CaptureController() -- Click
      game:GetService'VirtualUser':Button1Down(Vector2.new(1270, 371))

   until pk == false
   plr2.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
   if pk == false then
      if SecondWorld then
         game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-385.250916, 73.0458984, 297.388397)
      else
         game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(995.820557, 16.5516472, 1433.27771)
      end
   end
end)

Pvp:addToggle("Kill Player (Yasuo)",false,function(bool)
   pk = bool

   local plr1 = game.Players.LocalPlayer.Character
   local plr2 = game.Players:FindFirstChild(KillPlayer)

   repeat wait()
      RandomLOL = math.random(-20,20)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(RandomLOL, RandomLOL, RandomLOL)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(RandomLOL, RandomLOL, RandomLOL)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(RandomLOL, RandomLOL, RandomLOL)
      end)

      spawn(function()
         plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(RandomLOL, RandomLOL, RandomLOL)
      end)

      plr2.Character.HumanoidRootPart.Size = Vector3.new(50, 50, 50)

      game:GetService'VirtualUser':CaptureController() -- Click
      game:GetService'VirtualUser':Button1Down(Vector2.new(1270, 371))

   until pk == false
   plr2.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
   if pk == false then
      if SecondWorld then
         game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-385.250916, 73.0458984, 297.388397)
      else
         game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(995.820557, 16.5516472, 1433.27771)
      end
   end
end)

spawn(function()
  while wait(.1) do
     for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do 
       if v:IsA("Tool") then
         if v:FindFirstChild("RemoteFunctionShoot") then  
           SelectToolWeaponGun = v.Name
         end
       end
     end
  end
end)

Pvp:addToggle("Kill Player (Gun)",false,function(bool)
   pk = bool
   local plr1 = game.Players.LocalPlayer.Character
   local plr2 = game.Players:FindFirstChild(KillPlayer)
   if pk == true then
   repeat wait()
         zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
         RandomGun = math.random(1,4)
         spawn(function()
         if plr2.Character:FindFirstChild("HumanoidRootPart") ~= nil then
            plr1.HumanoidRootPart.CFrame = plr2.Character.HumanoidRootPart.CFrame * CFrame.new(30, 85, 0)
         end
      end)
         plr2.Character.HumanoidRootPart.Size = Vector3.new(70, 70, 70)		
         game:GetService'VirtualUser':CaptureController() -- Click
         game:GetService'VirtualUser':Button1Down(Vector2.new(1270, 371))
   until pk == false
   end
   plr2.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
   if SecondWorld then
      game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-385.250916, 73.0458984, 297.388397)
   else
      game.Players.localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(995.820557, 16.5516472, 1433.27771)
   end
end)

Pvp:addToggle("Silent Aim Gun",false,function(bool)
   if KillPlayer == "" and bool then
   elseif bool then 
      Aimbot = bool
   end
end)

Pvp:addToggle("AutoAim Skill",false,function(bool)
  Skill = bool
  while Skill do 
  wait()
    pcall(function()
      if game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool") and game.Players.LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name]:FindFirstChild("MousePos") then
         local args = {
            [1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position}
         game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name].RemoteEvent:FireServer(unpack(args))

         local args = {
            [1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position}
         game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name].RemoteEvent:FireServer(unpack(args))

         local args = {
            [1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position}
         game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name].RemoteEvent:FireServer(unpack(args))

         local args = {
            [1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position}
         game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name].RemoteEvent:FireServer(unpack(args))

         local args = {
            [1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position}
         game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name].RemoteEvent:FireServer(unpack(args))

         local args = {
            [1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position}
         game:GetService("Players").LocalPlayer.Character[game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name].RemoteEvent:FireServer(unpack(args))
      end
    end)
  end
end)

local lp = game:GetService('Players').LocalPlayer
local mouse = lp:GetMouse()
mouse.Button1Down:Connect(function()
  if  Aimbot and game.Players.LocalPlayer.Character:FindFirstChild(SelectToolWeaponGun) ~= nil then
     spawn(function()
        local args = {[1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position, [2] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart}
        if game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart ~= nil and game:GetService("Players").LocalPlayer.Character:FindFirstChild(SelectToolWeaponGun) and game:GetService("Players").LocalPlayer.Character:FindFirstChild(SelectToolWeaponGun).RemoteFunctionShoot:InvokeServer(unpack(args)) then
        end
     end)
     spawn(function()
      if KillPlayer then
        local args = {[1] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart.Position, [2] = game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart}
        if game:GetService("Players"):FindFirstChild(KillPlayer).Character.HumanoidRootPart~= nil and game:GetService("Players").LocalPlayer.Character:FindFirstChild(SelectToolWeaponGun) and game:GetService("Players").LocalPlayer.Character:FindFirstChild(SelectToolWeaponGun).RemoteFunctionShoot:InvokeServer(unpack(args)) then
        end
      end
     end)
  end 
end)

Pvp:addToggle("Spectate Player",false,function(bool)
   Sp = bool
   if game.Players:FindFirstChild(KillPlayer) ~= nil then
      local plr1 = game.Players.LocalPlayer.Character.Humanoid
      local plr2 = game.Players:FindFirstChild(KillPlayer)
      repeat wait(.1)
         if plr2 and plr2.Name ~= "amprodreamxd"then
            if plr2.Character:FindFirstChild("Humanoid") ~= nil then
               wait(0.1)
               if plr2.Character:FindFirstChild("Humanoid") ~= nil then
                  game.Workspace.Camera.CameraSubject = plr2.Character.Humanoid
               end
            end
         end
      until Sp == false 
   game.Workspace.Camera.CameraSubject = game.Players.LocalPlayer.Character.Humanoid
end

end)
Pvp:addButton("Teleport Player",function()
   if game.Players:FindFirstChild(KillPlayer) ~= nil then
   local plr1 = game.Players.LocalPlayer.Character
   
   local plr2 = game.Players:FindFirstChild(KillPlayer)
   TeleportTween(plr2.Character.HumanoidRootPart.CFrame,TweenSpeed)
   end
end)
Pvp:addButton("Check All Player Stats",function()
   if KillPlayer ~= "" or KillPlayer ~= nil then
   local Level = game:GetService("Players")[KillPlayer].Data.Level.Value
   lib:Notify("Level:",Level)
   wait(1)
   local Beli = game:GetService("Players")[KillPlayer].Data.Beli.Value
   lib:Notify("Beli:",Beli)
   wait(1)
   local F = game:GetService("Players")[KillPlayer].Data.Fragments.Value
   lib:Notify("Fragments:",F)
   wait(1)
   local Race = game:GetService("Players")[KillPlayer].Data.Race.Value
   lib:Notify("Race:",Race)
   wait(1)
   local DF = game:GetService("Players")[KillPlayer].Data.DevilFruit.Value
   lib:Notify("Devil Fruit:",DF)
   wait(1)
   local s = game:GetService("Players")[KillPlayer].Data.SpawnPoint.Value
   lib:Notify("Check SpawnPoint:",s)
   wait(1)
   local BH = game:GetService("Players")[KillPlayer].leaderstats["Bounty/Honor"].Value
   lib:Notify("Bounty/Honor:",BH)
   end
end)
Pvp:addToggle("HitBox Toggle",false,function(v)
   Hitboxv = v
   while Hitboxv do
      wait()
      if game.Players:FindFirstChild(KillPlayer) ~= nil and game.Players:FindFirstChild(KillPlayer).Name ~= "AnIslandAI" and game.Players:FindFirstChild(KillPlayer) ~= "amprodreamxd" then
         local plr2 = game.Players:FindFirstChild(KillPlayer)
         plr2.Character.HumanoidRootPart.Size = Vector3.new(Hitboxi,Hitboxi,Hitboxi)
      end
   end
end)

Pvp:addSlider("HitBox Extender",10,1,60,function(v)
   Hitboxi = v
 end)

Pvp:addButton("Get Kill Player Quest",function()
  local args = {[1] = "PlayerHunter"}
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)


local LocalPlr = Players:addSection("Local Player")
LocalPlr:addButton("Ghost Player (OP)",function() -- Ghost
   local name = game.Players.LocalPlayer
   _G.name = name
   local namex = tostring(_G.name)
   for i,v in pairs(game.Workspace.Characters[namex]:GetChildren()) do
      if v.Name == "LowerTorso" then   
       v:Destroy()
      end
   end
end)

LocalPlr:addToggle("Remove Crew & Marine",false,function(t)
   getgenv().RemoveCrew = t
   if getgenv().RemoveCrew == false then return end 
      while getgenv().RemoveCrew do 
         wait(.1)
      
      local name = game.Players.LocalPlayer
      _G.name = name
      local namex = tostring(_G.name)
      for i,v in pairs(game:GetService("Workspace").Characters[namex].HumanoidRootPart:GetChildren()) do
         if v.Name == "CrewBBG" then   
            v:Destroy()
         end
      end
      for i,v in pairs(game:GetService("Workspace").Characters[namex].HumanoidRootPart:GetChildren()) do
         if v.Name == "MarineBBG" then  
            v:Destroy()
         end
      end
   end
end)

LocalPlr:addSlider("Zoom Distance",0,200,5000,function(value)
   game.Players.LocalPlayer.CameraMaxZoomDistance = value
end)

LocalPlr:addToggle("Set Range Obversation",getgenv().IRO,function(t)
   getgenv().IRO = t 
   local VS = game.Players.LocalPlayer.VisionRadius.Value
   RO = getgenv().RangeO
   while getgenv().IRO do
      wait()
      local player = game.Players.LocalPlayer
      local char = player.Character
      local VisionRadius = player.VisionRadius
      if player then
         if char.Humanoid.Health <= 0 then 
            wait(5) 
         end
         VisionRadius.Value = getgenv().RangeO
      elseif getgenv().IRO == false and player then
         VisionRadius.Value = VS
      end
   end
end)

LocalPlr:addSlider("Range Obversation",getgenv().RangeO,1000,50000,function(t)
   getgenv().RangeO = t
end)

LocalPlr:addToggle("No Stun",getgenv().NoStun,function(t)
   getgenv().NoStun = t
   while getgenv().NoStun do
      wait()
      local player = game.Players.LocalPlayer
      local char = player.Character
      if char then
         if char.Humanoid.Health <= 0 then 
            wait(5)
         end
         local Stun = char.Stun
         Stun.Value = 0
      end
   end
end)

LocalPlr:addButton("Inf Geppo",function()
   getgenv().InfGeppo = true
   NoGeppoCD()
end)

LocalPlr:addButton("Dash No CD",function()
   getgenv().NoDashCD = true
   NoDashCD()
end)

LocalPlr:addButton("Soru No CD",function()
   getgenv().NoSoruCD = true
   NoSoruCD()
end)

function NoGeppoCD()
	if getgenv().InfGeppo then
		for i,v in next, getgc() do
			if game.Players.LocalPlayer.Character.Geppo then
				if typeof(v) == "function" and getfenv(v).script == game.Players.LocalPlayer.Character.Geppo then
					for i2,v2 in next, getupvalues(v) do
						if tostring(i2) == "9" then
							repeat wait(.1)
								setupvalue(v,i2,0)
							until not getgenv().InfGeppo or game.Players.LocalPlayer.Character.Humanoid.Health <= 0 
                     getgenv().InfGeppo = false
						end
					end
				end
			end
		end
	end
end

function NoSoruCD()
	if getgenv().NoSoruCD and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") ~= nil  then
		for i,v in next, getgc() do
			if game.Players.LocalPlayer.Character.Soru then
				if typeof(v) == "function" and getfenv(v).script == game.Players.LocalPlayer.Character.Soru then
					for i2,v2 in next, getupvalues(v) do
						if typeof(v2) == "table"  then
							repeat wait(.1)
								v2.LastUse = 0
							until not getgenv().NoSoruCD or game.Players.LocalPlayer.Character.Humanoid.Health <= 0
                     getgenv().NoSoruCD = false
						end
					end
				end
			end
		end
	end
end

function NoDashCD()
   if getgenv().NoDashCD then
      for i,v in next, getgc() do
            if game.Players.LocalPlayer.Character.Dodge then
               if typeof(v) == "function" and getfenv(v).script == game.Players.LocalPlayer.Character.Dodge then
               for i2,v2 in next, getupvalues(v) do
                  if tostring(v2) == "0.4"  then
                     repeat wait(.1)
                        setupvalue(v,i2,0)
                     until not getgenv().NoDashCD or game.Players.LocalPlayer.Character.Humanoid.Health <= 0 
                     getgenv().NoDashCD = false
                  end
               end
            end
         end
      end
   end
end
NoDashCD()

wait()

local Players = Players:addSection("Players")



local Dungeon = lib:addPage("Dungeon", 7061409730)

if MyDevilFruit == "Rumble-Rumble" then AutoChip = "Rumble"
elseif MyDevilFruit == "Flame-Flame" then AutoChip = "Flame"
elseif MyDevilFruit == "Quake-Quake" then AutoChip = "Quake"
elseif MyDevilFruit == "Dark-Dark" then AutoChip = "Dark"
elseif MyDevilFruit == "String-String" then AutoChip = "String"
elseif MyDevilFruit == "Magma-Magma" then AutoChip = "Magma"
elseif MyDevilFruit == "Light-Light" then AutoChip = "Light"
elseif MyDevilFruit == "Human-Human: Buddha" then AutoChip = "Human: Buddha"
else AutoChip = "Ice"
end
local Chip = Dungeon:addSection("Buy Chip is "..AutoChip.."")

Chip:addButton("Buy Auto Chip",function(x)
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("RaidsNpc", "Select", AutoChip)
end)

Chip:addDropdown("Select Buy Microchip", {"Flame","Ice","Quake","Light","Dark","String","Rumble"}, function(Value)
   Buychips = Value
end)

Chip:addButton(" Buy / trade Microchip",function()

   local args = {
   [1] = "RaidsNpc",
   [2] = "Check"
   }
   wait(.1)
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

   local args = {
   [1] = "RaidsNpc",
   [2] = "Select",
   [3] = Buychips
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Chip:addButton("Law Raid Chip",function()
   local args = {
      [1] = "BlackbeardReward",
      [2] = "Microchip",
      [3] = "2"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)
local Raid = Dungeon:addSection("Raid")
Raid:addToggle("Fully Auto Raid",false,function(Value)
   FullyAutoRaid = Value
   killaura = Value
   NextIsland = Value
   while FullyAutoRaid do 
      wait(0.1)
      spawn(function()
         zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
      end)
      local args = {[1] = "Cousin", [2] = "Buy"}
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      wait(0.5)
      if Buychips ~= nil and not AutoRaidJoined then
         local args = {
            [1] = "RaidsNpc",
            [2] = "Select",
            [3] = Buychips
            }
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
            wait(0.5)
         for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
            if string.find(v.Name,"Microchip") then
               if not AutoRaidJoined then
                  if ThirdWorld then
                     fireclickdetector(game:GetService("Workspace").Map["Boat Castle"].RaidSummon2.Button.Main.ClickDetector, 1)
                  elseif SecondWorld then
                     fireclickdetector(game:GetService("Workspace").Map.CircleIsland.RaidSummon2.Button.Main.ClickDetector, 1)
                  end
                     AutoRaidJoined = true
               else AutoRaidJoined = true
               end
            end
         end
         wait(25)
      repeat wait() until game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == false
      wait(5)
      AutoRaidJoined = false
      lib:Notify("FullyAutoRaid","Another Raid or Off")
      else lib:Notify("FullyAutoRaid","Please Select A Chip")
      end
   end
end)
Raid:addToggle("Auto Join Raid",false,function(Value)
   AutoJoinRaid = Value
   while AutoJoinRaid do 
      wait(0.1)
      for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
         if string.find(v.Name,"Microchip") then
            if ThirdWorld and not AutoRaidJoined  then
               fireclickdetector(game:GetService("Workspace").Map["Boat Castle"].RaidSummon2.Button.Main.ClickDetector, 1)
               AutoRaidJoined = true
               wait(25)
               repeat wait() until game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == false
               wait(5)
               AutoRaidJoined = false
            elseif SecondWorld and not AutoRaidJoined then
               fireclickdetector(game:GetService("Workspace").Map.CircleIsland.RaidSummon2.Button.Main.ClickDetector, 1)
               AutoRaidJoined = true 
               wait(25)
               repeat wait() until game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == false
               wait(5)
               AutoRaidJoined = false
            end
         end
      end
   end
end)

Raid:addToggle("Kill Aura",false,function(Value)
   killaura = Value
end)

Raid:addToggle("Next Island",false,function(value)
        
   if SecondWorld or ThirdWorld then
      NextIsland = value
         if NextIsland == true then

         _G.conn = game:GetService("RunService").Stepped:Connect(function() 
            zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
         end)

         else
          _G.conn:Disconnect() 
         end
   elseif FirstWorld then
   end
end)

Raid:addToggle("Auto Awakener",false,function(value)
   if SecondWorld then
      Awakener = value
      elseif FirstWorld then
   end
end)

Raid:addButton("Teleport to Dungeon",function()
   if SecondWorld then
      TeleportTween(CFrame.new(-6438.73535, 250.645355, -4501.50684),TweenSpeed)
   elseif ThirdWorld then
      TeleportTween(CFrame.new(-5034.16309, 314.9664, -2948.1499, 0.899162233, -3.93972392e-08, 0.437615454, 3.5941877e-08, 1, 1.61778075e-08, -0.437615454, 1.18224841e-09, 0.899162233),TweenSpeed)
   elseif FirstWorld then
   end
end)

if SecondWorld then
   Raid:addButton("Teleport to Lab",function()
      TeleportTween(CFrame.new(-6438.73535, 250.645355, -4501.50684),TweenSpeed)
   end)
end

Raid:addButton("Awakening Room",function()
   if SecondWorld then
      TeleportTween(CFrame.new(266.227783, 1.39509034, 1857.00732),TweenSpeed)
   elseif FirstWorld then
   end
end)

spawn(function()
   while wait(1) do
      if Awakener then
         local args = {[1] = "Awakener",[2] = "Check"}
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

         local args = {[1] = "Awakener",[2] = "Awaken"}
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end
   end
end)

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      if NextIsland then
         if game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 5") and game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == true then
            Island5CFrame = game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 5").CFrame*CFrame.new(0,50,0)
            TeleportTween(Island5CFrame,TweenSpeed)
         elseif game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 4") and game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == true then
            Island4CFrame = game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 4").CFrame*CFrame.new(0,50,0)
            TeleportTween(Island4CFrame,TweenSpeed)
         elseif game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 3") and game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == true then
            Island3CFrame = game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 3").CFrame*CFrame.new(0,50,0)
            TeleportTween(Island3CFrame,TweenSpeed)
         elseif game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 2") and game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == true then
            Island2CFrame = game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 2").CFrame*CFrame.new(0,50,0)
            TeleportTween(Island2CFrame,TweenSpeed)
         elseif game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 1") and game:GetService("Players").LocalPlayer.PlayerGui.Main.Timer.Visible == true then
            Island1CFrame = game:GetService("Workspace")["_WorldOrigin"].Locations:FindFirstChild("Island 1").CFrame*CFrame.new(0,50,0)
            TeleportTween(Island1CFrame,TweenSpeed)
         end
      end
   end
end)

     


wait()
local Misc = lib:addPage("Misc", 6034900727)
local RSFunction = Misc:addSection("Misc")
local LocalPlayer = game:GetService'Players'.LocalPlayer

RSFunction:addButton("Join Pirates Team",function()
  local args = {[1] = "SetTeam",[2] = "Pirates"}
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args)) 
  local args = {
     [1] = "BartiloQuestProgress"
  }
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
  local args = {
     [1] = "Buso"
  }
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

RSFunction:addButton("Join Marines Team",function()
  local args = {
     [1] = "SetTeam",
     [2] = "Marines"
  }
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

  local args = {
     [1] = "BartiloQuestProgress"
  }

  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
  local args = {
     [1] = "Buso"
  }
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

end)
local RigC = require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework) --require CDAttack module
local CD = 0 -- time to next attack
function stopweaponcd()
   pcall(function()
      RigC.activeController.attacking = false 
      RigC.activeController.active = false
      RigC.activeController.timeToNextAttack = CD
   end)
 end
spawn(function()
   while game:GetService("RunService").RenderStepped:wait(0.05) do
     if getgenv().NoCDAttack then
       stopweaponcd()
     end
   end
 end)
RSFunction:addToggle("No Cd Attack",false,function(Value)
   getgenv().NoCDAttack = Value
end)
RSFunction:addToggle("Walk on Water",false,function(Value)
   _G.WalkWater = Value
   if _G.WalkWater == true then
      game.Players.LocalPlayer.Data.DevilFruit.Value = ("Ice-Ice")
   elseif _G.WalkWater == false then
      game.Players.LocalPlayer.Data.DevilFruit.Value = (MyDevilFruit)
   end
end)
RSFunction:addButton("Open Fruits Shop",function()
   game:GetService("Players")["LocalPlayer"].PlayerGui.Main.FruitShop.Visible = true
end)

RSFunction:addButton("Teleport And Open Inventory",function()
   TeleportTween(CFrame.new(-216.974274, 6.72994137, 5324.96387, 0.987430871, -8.22794721e-09, 0.158051461, 1.06244979e-09, 1, 4.54209754e-08, -0.158051461, -4.46821531e-08, 0.987430871),TweenSpeed)
   wait()
   game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Inventory.Visible = true
end)

RSFunction:addButton("Open Title",function()
   game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Titles.Visible = true
end)

RSFunction:addButton("Open Color",function()
   game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Colors.Visible = true
end)

RSFunction:addButton("Open Awakening Toggle",function()
  game:GetService("Players")["LocalPlayer"].PlayerGui.Main.AwakeningToggler.Visible = true
end)


function isnil(thing)
   return (thing == nil)
end
local function round(n)
   return math.floor(tonumber(n) + 0.5)
end
Number = math.random(1, 1000000)
function UpdatePlayerChams()
   for i,v in pairs(game:GetService'Players':GetChildren()) do
      pcall(function()
         if not isnil(v.Character) then
            if ESPPlayer then
               if not isnil(v.Character.Head) and not v.Character.Head:FindFirstChild('NameEsp'..Number) then
                  local bill = Instance.new('BillboardGui',v.Character.Head)
                  bill.Name = 'NameEsp'..Number
                  bill.ExtentsOffset = Vector3.new(0, 1, 0)
                  bill.Size = UDim2.new(1,200,1,30)
                  bill.Adornee = v.Character.Head
                  bill.AlwaysOnTop = true
                  local name = Instance.new('TextLabel',bill)
                  name.Font = "GothamBold"
                  name.FontSize = "Size14"
                  name.TextWrapped = true
                  name.Text = (v.Name ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Character.Head.Position).Magnitude/3) ..' M')
                  name.Size = UDim2.new(1,0,1,0)
                  name.TextYAlignment = 'Top'
                  name.BackgroundTransparency = 1
                  name.TextStrokeTransparency = 0.5
                  if v.Team == game.Players.LocalPlayer.Team then
                     name.TextColor3 = Color3.new(0,255,0)
                  else
                     name.TextColor3 = Color3.new(255,0,0)
                  end
               else
                  v.Character.Head['NameEsp'..Number].TextLabel.Text = (v.Name ..'   \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Character.Head.Position).Magnitude/3) ..' M')
               end
            else
               if v.Character.Head:FindFirstChild('NameEsp'..Number) then
                  v.Character.Head:FindFirstChild('NameEsp'..Number):Destroy()
               end
            end
         end
      end)
   end
end

function UpdateChestChams() 
   for i,v in pairs(game.Workspace:GetChildren()) do
      pcall(function()
         if string.find(v.Name,"Chest") then
            if ChestESP then
               if string.find(v.Name,"Chest") then
                  if not v:FindFirstChild('NameEsp'..Number) then
                     local bill = Instance.new('BillboardGui',v)
                     bill.Name = 'NameEsp'..Number
                     bill.ExtentsOffset = Vector3.new(0, 1, 0)
                     bill.Size = UDim2.new(1,200,1,30)
                     bill.Adornee = v
                     bill.AlwaysOnTop = true
                     local name = Instance.new('TextLabel',bill)
                     name.Font = "GothamBold"
                     name.FontSize = "Size14"
                     name.TextWrapped = true
                     name.Size = UDim2.new(1,0,1,0)
                     name.TextYAlignment = 'Top'
                     name.BackgroundTransparency = 1
                     name.TextStrokeTransparency = 0.5
                     if v.Name == "Chest1" then
                        name.TextColor3 = Color3.fromRGB(109, 109, 109)
                        name.Text = ("Chest 1" ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
                     end
                     if v.Name == "Chest2" then
                        name.TextColor3 = Color3.fromRGB(173, 158, 21)
                        name.Text = ("Chest 2" ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
                     end
                     if v.Name == "Chest3" then
                        name.TextColor3 = Color3.fromRGB(85, 255, 255)
                        name.Text = ("Chest 3" ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
                     end
                  else
                     v['NameEsp'..Number].TextLabel.Text = (v.Name ..'   \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
                  end
               end
            else
               if v:FindFirstChild('NameEsp'..Number) then
                  v:FindFirstChild('NameEsp'..Number):Destroy()
               end
            end
         end
      end)
   end
end

function UpdateDevilChams() 
   for i,v in pairs(game.Workspace:GetChildren()) do
      pcall(function()
         if DevilFruitESP then
            if string.find(v.Name, "Fruit") then   
               if not v.Handle:FindFirstChild('NameEsp'..Number) then
                  local bill = Instance.new('BillboardGui',v.Handle)
                  bill.Name = 'NameEsp'..Number
                  bill.ExtentsOffset = Vector3.new(0, 1, 0)
                  bill.Size = UDim2.new(1,200,1,30)
                  bill.Adornee = v.Handle
                  bill.AlwaysOnTop = true
                  local name = Instance.new('TextLabel',bill)
                  name.Font = "GothamBold"
                  name.FontSize = "Size14"
                  name.TextWrapped = true
                  name.Size = UDim2.new(1,0,1,0)
                  name.TextYAlignment = 'Top'
                  name.BackgroundTransparency = 1
                  name.TextStrokeTransparency = 0.5
                  name.TextColor3 = Color3.fromRGB(255, 0, 0)
                  name.Text = (v.Name ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Handle.Position).Magnitude/3) ..' M')
               else
                  v.Handle['NameEsp'..Number].TextLabel.Text = (v.Name ..'   \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Handle.Position).Magnitude/3) ..' M')
               end
            end
         else
            if v.Handle:FindFirstChild('NameEsp'..Number) then
               v.Handle:FindFirstChild('NameEsp'..Number):Destroy()
            end
         end
      end)
   end
end

function UpdateFlowerChams() 
   for i,v in pairs(game.Workspace:GetChildren()) do
      pcall(function()
         if v.Name == "Flower2" or v.Name == "Flower1" then
            if FlowerESP then 
               if not v:FindFirstChild('NameEsp'..Number) then
                  local bill = Instance.new('BillboardGui',v)
                  bill.Name = 'NameEsp'..Number
                  bill.ExtentsOffset = Vector3.new(0, 1, 0)
                  bill.Size = UDim2.new(1,200,1,30)
                  bill.Adornee = v
                  bill.AlwaysOnTop = true
                  local name = Instance.new('TextLabel',bill)
                  name.Font = "GothamBold"
                  name.FontSize = "Size14"
                  name.TextWrapped = true
                  name.Size = UDim2.new(1,0,1,0)
                  name.TextYAlignment = 'Top'
                  name.BackgroundTransparency = 1
                  name.TextStrokeTransparency = 0.5
                  name.TextColor3 = Color3.fromRGB(255, 0, 0)
                  if v.Name == "Flower1" then 
                     name.Text = ("Blue Flower" ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
                     name.TextColor3 = Color3.fromRGB(0, 0, 255)
                  end
                  if v.Name == "Flower2" then
                     name.Text = ("Red Flower" ..' \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
                     name.TextColor3 = Color3.fromRGB(255, 0, 0)
                  end
               else
                  v['NameEsp'..Number].TextLabel.Text = (v.Name ..'   \n'.. round((game:GetService('Players').LocalPlayer.Character.Head.Position - v.Position).Magnitude/3) ..' M')
               end
            else
               if v:FindFirstChild('NameEsp'..Number) then
                  v:FindFirstChild('NameEsp'..Number):Destroy()
               end
            end
         end   
      end)
   end
end

RSFunction:addToggle("ESP Player",false,function(a)
   ESPPlayer = a
   while ESPPlayer do wait()
      UpdatePlayerChams()
   end
end)

RSFunction:addToggle("ESP Chest",false,function(a)
   ChestESP = a
   while ChestESP do wait()
      UpdateChestChams() 
   end
end)

RSFunction:addToggle("ESP Devil Fruit",false,function(a)
   DevilFruitESP = a
   while DevilFruitESP do wait()
      UpdateDevilChams() 
   end
end)

RSFunction:addToggle("ESP Flower",false,function(a)
   FlowerESP = a

   while FlowerESP do wait()
      UpdateFlowerChams() 

   end
end)

RSFunction:addButton("Teleport Haki Pad",function()
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("activateColor","Winter Sky")
   wait(0.5)
   TeleportTween(CFrame.new(-5420.16602, 1084.9657, -2666.8208, 0.390717864, 0, 0.92051065, 0, 1, 0, -0.92051065, 0, 0.390717864),TweenSpeed)
   wait(0.5)
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("activateColor","Pure Red")
   wait(0.5)
   TeleportTween(CFrame.new(-5414.41357, 309.865753, -2212.45776, 0.374604106, -0, -0.92718488, 0, 1, -0, 0.92718488, 0, 0.374604106),TweenSpeed)
   wait(0.5)
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("activateColor","Snow White")
   wait(0.5)
   TeleportTween(CFrame.new(-4971.47559, 335.565765, -3720.02954, -0.92051065, 0, 0.390717506, 0, 1, 0, -0.390717506, 0, -0.92051065),TweenSpeed)
end)
RSFunction:addButton("Teleport Holy Torch",function()
   if not game.Players.LocalPlayer.Backpack:FindFirstChild("Holy Torch") or not game.Players.LocalPlayer.Character:FindFirstChild("Holy Torch") then
      wait(1)
      TeleportTween(CFrame.new(5154.17676, 141.786423, 911.046326, 0.0786230937, 1.10107962e-07, 0.996904433, 8.76694628e-09, 1, -1.11141297e-07, -0.996904433, 1.74780794e-08, 0.0786230937),TweenSpeed)
      wait(0.2)
      TeleportTween(CFrame.new(5148.03613, 162.352493, 910.548218, 0, 0, -1, 0, 1, 0, 1, 0, 0),TweenSpeed)
      EquipWeapon("Holy Torch")
      TweenSpeed = 300
      wait(2)
      TeleportTween(CFrame.new(-10752.7695, 412.229523, -9366.36328, 0.97063309, -3.15659037e-08, 0.240564749, 5.60982159e-08, 1, -9.51298205e-08, -0.240564749, 1.05831404e-07, 0.97063309),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-11673.4111, 331.749023, -9474.34668, 0.200540408, 6.91700279e-08, -0.979685426, -2.19707559e-08, 1, 6.6106935e-08, 0.979685426, 8.2673175e-09, 0.200540408),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-12133.3389, 519.47522, -10653.1904, 0.845298052, -6.34159107e-08, -0.534295022, 4.46860327e-08, 1, -4.79938898e-08, 0.534295022, 1.66936172e-08, 0.845298052),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-13336.5, 485.280396, -6983.35254, 0.912677109, -1.29153763e-08, -0.408681422, 2.20926388e-08, 1, 1.77352568e-08, 0.408681422, -2.52154138e-08, 0.912677109),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-13487.4131, 334.84845, -7926.34863, -0.999999702, -6.9906867e-09, -0.000721037388, -6.9868662e-09, 1, -5.30099697e-09, 0.000721037388, -5.29595745e-09, -0.999999702),TweenSpeed)
      TweenSpeed = 200
   else
      EquipWeapon("Holy Torch")
      TweenSpeed = 300
      wait(2)
      TeleportTween(CFrame.new(-10752.7695, 412.229523, -9366.36328, 0.97063309, -3.15659037e-08, 0.240564749, 5.60982159e-08, 1, -9.51298205e-08, -0.240564749, 1.05831404e-07, 0.97063309),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-11673.4111, 331.749023, -9474.34668, 0.200540408, 6.91700279e-08, -0.979685426, -2.19707559e-08, 1, 6.6106935e-08, 0.979685426, 8.2673175e-09, 0.200540408),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-12133.3389, 519.47522, -10653.1904, 0.845298052, -6.34159107e-08, -0.534295022, 4.46860327e-08, 1, -4.79938898e-08, 0.534295022, 1.66936172e-08, 0.845298052),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-13336.5, 485.280396, -6983.35254, 0.912677109, -1.29153763e-08, -0.408681422, 2.20926388e-08, 1, 1.77352568e-08, 0.408681422, -2.52154138e-08, 0.912677109),TweenSpeed)
      wait(0.4)
      TeleportTween(CFrame.new(-13487.4131, 334.84845, -7926.34863, -0.999999702, -6.9906867e-09, -0.000721037388, -6.9868662e-09, 1, -5.30099697e-09, 0.000721037388, -5.29595745e-09, -0.999999702),TweenSpeed)
      TweenSpeed = 200
   end
end)



RSFunction:addToggle("Auto Click",false,function(value)
   AuctoClick = value
   getgenv().C = value
   if getgenv().C == false then return end 
   while getgenv().C do wait()
      game:GetService'VirtualUser':CaptureController()
      game:GetService'VirtualUser':Button1Down(Vector2.new(1240, 372))
      game:GetService'VirtualUser':CaptureController()
      game:GetService'VirtualUser':Button1Down(Vector2.new(1230, 652))
      game:GetService'VirtualUser':CaptureController()
      game:GetService'VirtualUser':Button1Down(Vector2.new(1250, 252))
      game:GetService'VirtualUser':CaptureController()
      game:GetService'VirtualUser':Button1Down(Vector2.new(1260, 282))
   end
end)

RSFunction:addTextbox("Change the message!!", false, function(m)
   chat = m
end)

RSFunction:addToggle("Spam Message!!",false,function(s)
   _G.A = s
   if _G.A == false then return end 
   while _G.A do wait(.1)
      local ohString1 = chat 
      local ohString2 = "All"
      game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(ohString1, ohString2)
   end
end)



RSFunction:addToggle("Highlight Mode",false,function(t)
   getgenv().HighlightMode = t
   if getgenv().HighlightMode  == false then return end 
   while getgenv().HighlightMode do wait()
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Beli.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.HP.Visible = true
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Energy.Visible = true
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.StatsButton.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.ShopButton.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Skills.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Level.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.MenuButton.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Code.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Settings.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Mute.Visible = false
      game:GetService("Players")["LocalPlayer"].PlayerGui.Main.CrewButton.Visible = false
      if not getgenv().HighlightMode  then
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Beli.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.HP.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Energy.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.StatsButton.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.ShopButton.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Skills.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Level.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.MenuButton.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Code.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Settings.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Mute.Visible = true
         game:GetService("Players")["LocalPlayer"].PlayerGui.Main.CrewButton.Visible = true
      end
   end
end)

ResetCode = {
   "THIRDSEA";
   "SUB2GAMERROBOT_RESET1";
   "Sub2UncleKizaru";
}

RSFunction:addButton("Redeem All Exp Code",function() 
  function UseCode(Text)
     game:GetService("ReplicatedStorage").Remotes.Redeem:InvokeServer(Text)
  end
  UseCode("FUDD10")
  UseCode("BIGNEWS")
  UseCode("THEGREATACE")
  UseCode("SUB2NOOBMASTER123")
  UseCode("Sub2Daigrock")
  UseCode("Axiore")
  UseCode("TantaiGaming")
  UseCode("STRAWHATMAINE")
  UseCode("Sub2OfficialNoobie")
  UseCode("UPD16")
  UseCode("SUB2GAMERROBOT_EXP1")
  UseCode("2BILLION")
end)

RSFunction:addDropdown("Redeem Reset Stats Code",ResetCode,function(v) 
   function UseCode(Text)
      game:GetService("ReplicatedStorage").Remotes.Redeem:InvokeServer(Text)
   end
   UseCode(v)
 end)

Fly = false  
speedSET = 25
function activatefly()
   local mouse=game.Players.LocalPlayer:GetMouse''
   localplayer=game.Players.LocalPlayer
   game.Players.LocalPlayer.Character:WaitForChild("HumanoidRootPart")
   local torso = game.Players.LocalPlayer.Character.HumanoidRootPart
   local keys={a=false,d=false,w=false,s=false}
   local e1
   local e2
   local function start()
      local pos = Instance.new("BodyPosition",torso)
      local gyro = Instance.new("BodyGyro",torso)
      pos.Name="EPIXPOS"
      pos.maxForce = Vector3.new(math.huge, math.huge, math.huge)
      pos.position = torso.Position
      gyro.maxTorque = Vector3.new(9e9, 9e9, 9e9)
      gyro.cframe = torso.CFrame
      repeat
         wait()
         localplayer.Character.Humanoid.PlatformStand=true
         local new=gyro.cframe - gyro.cframe.p + pos.position
         if not keys.w and not keys.s and not keys.a and not keys.d then
            speed=1
         end
         if keys.w then
            new = new + workspace.CurrentCamera.CoordinateFrame.lookVector * speed
            speed=speed+speedSET
         end
         if keys.s then
            new = new - workspace.CurrentCamera.CoordinateFrame.lookVector * speed
            speed=speed+speedSET
         end
         if keys.d then
            new = new * CFrame.new(speed,0,0)
            speed=speed+speedSET
         end
         if keys.a then
            new = new * CFrame.new(-speed,0,0)
            speed=speed+speedSET
         end
         if speed>speedSET then
            speed=speedSET
         end
         pos.position=new.p
         if keys.w then
            gyro.cframe = workspace.CurrentCamera.CoordinateFrame*CFrame.Angles(-math.rad(speed*90),0,0)
         elseif keys.s then
            gyro.cframe = workspace.CurrentCamera.CoordinateFrame*CFrame.Angles(math.rad(speed*90),0,0)
         else
            gyro.cframe = workspace.CurrentCamera.CoordinateFrame
         end
      until not Fly
      if gyro then 
         gyro:Destroy() 
      end
      if pos then 
         pos:Destroy() 
      end
      flying=false
      localplayer.Character.Humanoid.PlatformStand=false
      speed=0
   end
   e1=mouse.KeyDown:connect(function(key)
      if not torso or not torso.Parent then 
         flying=false e1:disconnect() e2:disconnect() return 
      end
      if key=="w" then
         keys.w=true
      elseif key=="s" then
         keys.s=true
      elseif key=="a" then
         keys.a=true
      elseif key=="d" then
         keys.d=true
      end
   end)
   e2=mouse.KeyUp:connect(function(key)
      if key=="w" then
         keys.w=false
      elseif key=="s" then
         keys.s=false
      elseif key=="a" then
         keys.a=false
      elseif key=="d" then
         keys.d=false
      end
   end)
   start()
end

RSFunction:addToggle("Fly (Broken)",false,function(Value)
  Fly = Value
  activatefly()
end)

RSFunction:addSlider("Fly Speed",speedSET,10,100,function(Value)
   speedSET = Value
end)

RSFunction:addToggle("No Clip",false,function(t)
  if t == true then
  _G.conn = game:GetService("RunService").Stepped:Connect(function()
     for _, v in pairs(game.Players.LocalPlayer.Character:GetDescendants()) do
        if v:IsA("BasePart") then
           v.CanCollide = false    
        end
     end
   end)
  else
   _G.conn:Disconnect()
  end
end)


RSFunction:addButton("Remove Lava",function()

  for i,v in pairs(game.Workspace:GetDescendants()) do
    if v.Name == "Lava" then   
      v:Destroy()
    end
  end

  for i,v in pairs(game.ReplicatedStorage:GetDescendants()) do
    if v.Name == "Lava" then   
      v:Destroy()
    end
  end
end)

RSFunction:addButton("Remove Water (Has damage)",function()
   for i,v in pairs(game.Workspace:GetDescendants()) do
      if v.Name == "Water;" then   
         v:Destroy()
      end
   end

   for i,v in pairs(game.ReplicatedStorage:GetDescendants()) do
      if v.Name == "Water;" then   
         v:Destroy()
      end
   end
end)

RSFunction:addButton("Remove Door (LAB)",function()
   if SecondWorld then
      for i,v in pairs(game:GetService("Workspace").Map.CircleIsland.Lab.Combo:GetDescendants()) do
         if v.Name == "Door" then   
            v:Destroy()
         end
      end
      for i,v in pairs(game.ReplicatedStorage:GetDescendants()) do
         if v.Name == "Lava" then   
            v:Destroy()
         end
      end
   elseif FirstWorld then
   end
end)
TweenSpeedChest = 300
RSFunction:addToggle("Auto Chest( Warn :Can get banned)",false,function(t)

_G.d = t
local function HttpGet(url)
    return game:GetService("HttpService"):JSONDecode(htgetf(url))
end
game:GetService('RunService').Stepped:connect(function()
if _G.d then
   zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
end
end)


function FindNearest(chests)
    local lowest = math.huge -- infinity
    local chest = nil
    for i,v in pairs(chests) do
        if v then
            local distance = (v.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).magnitude
            if distance < lowest then
                lowest = distance
                chest = v
            end
        end
    end
    return chest
end

game:GetService("ReplicatedStorage"):WaitForChild("Remotes")
local TeleportService = game:GetService("TeleportService")
while _G.d and wait() do
    local chests = {}
    for i,d in pairs(game:GetService("Workspace"):GetChildren()) do
        if string.find(d.Name, "Chest") ~= nil then
            table.insert(chests, d)
        end
    end
    if #chests == 0 then
        pcall(function()
            local d = HttpGet("https://www.roblox.com/games/getgameinstancesjson?placeId=" .. game.PlaceId .. "&startindex=0")
            local f = HttpGet("https://www.roblox.com/games/getgameinstancesjson?placeId=" .. game.PlaceId .. "&startindex=".. math.random(0,tonumber(d.TotalCollectionSize)))
            local c = f.Collection[math.random(1,#f.Collection)]
            game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, c.Guid)
        end)
        wait(0.5)
    end
if game.Players.LocalPlayer.Team == nil then 
game:GetService("ReplicatedStorage").Remotes["CommF_"]:InvokeServer("SetTeam", "Marines")
end
    if game.Players.LocalPlayer.Character then
        local close = FindNearest(chests)
if close == nil then
        if game.VIPServerOwnerId == 0 then
        pcall(function()
            local d = HttpGet("https://www.roblox.com/games/getgameinstancesjson?placeId=" .. game.PlaceId .. "&startindex=0")
            local f = HttpGet("https://www.roblox.com/games/getgameinstancesjson?placeId=" .. game.PlaceId .. "&startindex=".. math.random(0,tonumber(d.TotalCollectionSize)))
            local c = f.Collection[math.random(1,#f.Collection)]
            game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, c.Guid)
        end)
       end
        wait(0.5)
    else
        ChestCFrame = CFrame.new(close.CFrame.X,close.CFrame.Y,close.CFrame.Z)
    TeleportTween(ChestCFrame,TweenSpeedChest)
repeat wait() until d == nil or d.Parent == nil or _G.d == false
end
end
    
end
end)

wait()
local DevilFruit = lib:addPage("Devil Fruit", 6031488938)
local DF = DevilFruit:addSection("Your Fruit is: " ..MyDevilFruit.."")

DF:addToggle("Auto Buy Random Fruit",false,function(t)
  AutoBuyRandomFruit = t
   while AutoBuyRandomFruit do
      wait(1)
      local args = {[1] = "Cousin", [2] = "Buy"}
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
   end
end)

DF:addButton("Buy Random Fruit",function()
  local args = {[1] = "Cousin", [2] = "Buy"}
  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

DF:addToggle("Bring Fruit",false,function(t) 
   getgenv().BringFruit = t
   if getgenv().BringFruit == false then return end 
   while getgenv().BringFruit do wait(.1)
      for i,v in pairs(game.Workspace:GetChildren()) do -- fruitdrop
         if v:IsA "Tool" then
            getgenv().AutoElites = false
            if (v.Handle.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 800 then
               v.Handle.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
            else
               if AF == true then
                  AF = false
                  ChangeAF = true
               end
               if AFM == true then
                  AFM = false
                  ChangeAFM = true
               end
            oldCf = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
            TeleportTween(v.Handle.CFrame,TweenSpeed)
            wait(1)
            if (v.Handle.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 800 then
            v.Handle.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
            wait(1)
            if ChangeAF == true then
               AF = true
               ChangeAF = false
            end
            if ChangeAFM == true then
               AFM = true
               ChangeAFM = false
            end
            end
            end
         end
       end
   end
end)

fruitstock = {        
  "Bomb-Bomb",
  "Spike-Spike",
  "Chop-Chop",
  "Spring-Spring",
  "Kilo-Kilo",
  "Smoke-Smoke",
  "Spin-Spin",
  "Flame-Flame",
  "Falcon-Falcon",
  "Ice-Ice",
  "Sand-Sand",
  "Dark-Dark",
  "Diamond-Diamond",
  "Light-Light",
  "Love-Love",
  "Rubber-Rubber",
  "Barrier-Barrier",
  "Magma-Magma",
  "Door-Door",
  "Quake-Quake",
  "Human-Human: Buddha",
  "String-String",
  "Bird-Bird: Phoenix",
  "Rumble-Rumble",
  "Paw-Paw",
  "Gravity-Gravity",
  "Dough-Dough",
  "Venom-Venom",
  "Control-Control",
  "Dragon-Dragon"
}

SelectDevil = ""
Check = false

DF:addDropdown("Devil Fruit Sniper",fruitstock,function(ply) 
  SelectDevil = ply
end)

DF:addToggle("DevilFruit Sniper",false,function(value)
   if SelectDevil == "" and value then
      lib:Notify("DevilFruit Sniper","Select a Fruits")
   elseif value then
      BuyFruitSniper = vu
      BuyFruitSniper()
   end	
end)
DF:addToggle("Auto Store Fruits",false,function(value)
   AutoStoreFruit = value
   while AutoStoreFruit do
      wait(1)
      local args = {
         [1] = "getInventoryFruits"
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      AllFruit = fruitstock
      for i,v in pairs(game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))) do
         for i2,v2 in pairs(v) do
            FruitCheck = v2
            for i3,v3 in pairs(AllFruit) do
                  if v3 == v2 then
                     table.remove(AllFruit,i3)
                  end
            end
         end
         
      end
      for i,v in pairs(AllFruit) do
      local args = {
         [1] = "StoreFruit",
         [2] = v
      }

      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end
   end
end)
function BuyFruitSniper()
   while BuyFruitSniper do
      wait()
      local args = {[1] = "GetFruits"}
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "PurchaseRawFruit",
         [2] = SelectDevil
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
   end
end

wait()

local Shop = lib:addPage("Shop", 6035161534)
local StatsRe = Shop:addSection("Stats Reset")
StatsRe:addButton("Buy Stats Refund (2500 Fragment)",function()

   local args = {[1] = "BlackbeardReward",[2] = "Refund",[3] = "1"}
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
   local args = {[1] = "BlackbeardReward",[2] = "Refund",[3] = "2"}
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

end)

StatsRe:addButton("Buy Random Race",function()
   wait(.1)
   local args = {[1] = "BlackbeardReward",[2] = "Reroll",[3] = "2"}
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)
local Bones = Shop:addSection("Bones")
Bones:addButton("Buy Surprise",function()
   local args = {
      [1] = "Bones",
      [2] = "Check"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

   local args = {
      [1] = "Bones",
      [2] = "Buy",
      [3] = 1,
      [4] = 1
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Bones:addToggle("Auto Buy Surprise",false,function(v)
   AutoBuyBone = v
   while AutoBuyBone do
      local args = {
         [1] = "Bones",
         [2] = "Check"
      }
      if game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args)) >= 50 then

         local args = {
            [1] = "Bones",
            [2] = "Buy",
            [3] = 1,
            [4] = 1
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end
   end
end)
local Melee = Shop:addSection("Fighting Styles ✊")

Melee:addButton("Dark Step",function()
	local args = {
		[1] = "BuyBlackLeg"
	}
	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Melee:addButton("Electro",function()
	local args = {
	   [1] = "BuyElectro"
	}
	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

 Melee:addButton("Water Kung Fu",function()
	local args = {
	   [1] = "BuyFishmanKarate"
	}
	
	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

 Melee:addButton("Dragon Breath",function()
	local args = {
	   [1] = "BlackbeardReward",
	   [2] = "DragonClaw",
	   [3] = "1"
	}

	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
	local args = {
	   [1] = "BlackbeardReward",
	   [2] = "DragonClaw",
	   [3] = "2"
	}

	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

 Melee:addButton("Superhuman",function()
	local args = {
	   [1] = "BuySuperhuman"
	}
	
	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

 Melee:addButton("Death Step",function()
	local args = {
	   [1] = "BuyDeathStep"
	}
	
	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

 Melee:addButton("Sharkman Karate",function()
	local args = {
	   [1] = "BuySharkmanKarate",
	   [2] = true
	}

	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
	local args = {
	   [1] = "BuySharkmanKarate"
	}

	game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)
 Melee:addButton("Electric Claw",function()
   local args = {[1] = "BuyElectricClaw"}
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)
 Melee:addButton("Dragon Tailon",function()

local args = {
   [1] = "BuyDragonTalon",
   [2] = true
}

game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

local args = {
  [1] = "BuyDragonTalon"
}

game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

 end)


 local Abilities = Shop:addSection("Abilities ⚡️")

 Abilities:addButton("Skyjump",function()
   local args = {
     [1] = "BuyHaki",
     [2] = "Geppo"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

 Abilities:addButton("Enhancement",function()
   local args = {
     [1] = "BuyHaki",
     [2] = "Buso"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
 end)

Abilities:addButton("Flash Step",function()
   local args = {[1] = "BuyHaki",[2] = "Soru"}
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Abilities:addButton("Ken",function()
   local args = {[1] = "KenTalk",[2] = "Buy"}
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)
wait()

local Sword = Shop:addSection("Sword ⚔️")

Sword:addButton("Bisento",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Bisento"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Sword:addButton("Dual-Headed Blade",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Dual-Headed Blade"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Sword:addButton("Soul Cane",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Soul Cane"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Sword:addButton("Triple Katana",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Triple Katana"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Sword:addButton("Pipe",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Pipe"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Sword:addButton("Dual Katana",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Dual Katana"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Sword:addButton("Iron Mace",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Iron Mace"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

local Accessories = Shop:addSection("Accessories")

Accessories:addButton("Black Cape",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Black Cape"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Accessories:addButton("Swordsman Hat",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Swordsman Hat"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Accessories:addButton("Tomoe Ring",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Tomoe Ring"
   }
end)
local Gun = Shop:addSection("Gun 🔫")


Gun:addButton("Slingshot",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Slingshot"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Gun:addButton("Musket",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Musket"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Gun:addButton("Refined Slingshot",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Refined Slingshot"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Gun:addButton("Cannon",function()
   local args = {
      [1] = "BuyItem",
      [2] = "Cannon"
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

Gun:addButton("Bizarre Rifle",function()
   local args = {
      [1] = "Ectoplasm",
      [2] = "Buy",
      [3] = 1
   }
   game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end)

local Setting = lib:addPage("Setting", 6034509993)
local AutofarmSetting = Setting:addSection("AutoFarm Setting")

AutofarmSetting:addToggle("Fast Attack",getgenv().FastAttack, function(x)
    getgenv().FastAttack = x
end)

AutofarmSetting:addToggle("Autofarm : Magnet",getgenv().Magnet,function(Value)
	getgenv().Magnet = Value
   if getgenv().Magnet then
      Mode = "Magnet"
   else
      Mode = "Back"
   end
end)
AutofarmSetting:addToggle("AutoFarm Click",getgenv().AutoFarmClick,function(Value)
	getgenv().AutoFarmClick = Value
end)
AutofarmSetting:addSlider("Magnet Distance",getgenv().MagnetDistance,100,400,function(x)
    getgenv().MagnetDistance = x
end)

AutofarmSetting:addSlider("Fast Attack CD",getgenv().FastAttackCD,0,1,function(x)
   getgenv().FastAttackCD = x
end)
AutofarmSetting:addToggle("Refresh Mob if it took too long(Soon)",false,function(value)
end)

AutofarmSetting:addToggle("MobAura : Magnet",getgenv().MobAuraBring,function(value)
   getgenv().MobAuraBring = value
end)



local AutoFarmFruitSetting = Setting:addSection("AutoFarm Fruit Setting")

AutoFarmFruitSetting:addToggle("Skill Z",SkillZ,function(value)
   SkillZ = value
end)

AutoFarmFruitSetting:addSlider("Skill Z Hold Time",SkillZTime,0,4,function(value)
   SkillZTime = value
end)

AutoFarmFruitSetting:addToggle("Skill X",SkillX,function(value)
   SkillX = value
end)

AutoFarmFruitSetting:addSlider("Skill X Hold Time",SkillXTime,0,4,function(value)
   SkillXTime = value
end)

AutoFarmFruitSetting:addToggle("Skill C",SkillC,function(value)
   SkillC = value
end)

AutoFarmFruitSetting:addSlider("Skill X Hold Time",SkillCTime,0,4,function(value)
   SkillCTime = value
end)

AutoFarmFruitSetting:addToggle("Skill V",SkillV,function(value)
   SkillV = value
end)

AutoFarmFruitSetting:addSlider("Skill V Hold Time",SkillVTime,0,4,function(value)
   SkillVTime = value
end)

AutoFarmFruitSetting:addToggle("Skill F",SkillF,function(value)
   SkillF = value
end)

AutoFarmFruitSetting:addSlider("Skill F Hold Time",SkillFTime,0,4,function(value)
   SkillFTime = value
end)

local TeleportSetting = Setting:addSection("Teleport Setting")
TeleportSetting:addSlider("Tween Speed",TweenSpeed,100,400,function(v)
   TweenSpeed = v
end)

TeleportSetting:addSlider("Auto Farm Tween Speed",TweenSpeedF,100,400,function(v)
   TweenSpeedF = v
end)

TeleportSetting:addSlider("Auto Chest Tween Speed",TweenSpeedChest,100,400,function(v)
   TweenSpeedChest = v
end)

local GameSetting = Setting:addSection("Game Setting")

GameSetting:addKeybind("Hide UI", Enum.KeyCode.RightControl, function()
   lib:toggle()
end, function()

end)


for Setting, color in pairs(themes) do -- all in one theme changer, i know, im cool
	GameSetting:addColorPicker(Setting, color, function(color3)
		lib:setTheme(Setting, color3)
	end)
end
local filename1 = "NoAutoJoinDiscord.txt";
function SaveSetting1()
   print("Saving Data")
   local json;
   local HttpService = game:GetService("HttpService");
   if (writefile) then
      json = HttpService:JSONEncode(getgenv().NoAutoJoinDiscord)
      writefile(filename1, json);
   else
      print("Can't save because of your trash exploits")
   end
end
GameSetting:addButton("Turn on No Auto Join Discord",function()
   getgenv().NoAutoJoinDiscord = true
   SaveSetting1()
end)
local filename = "IslandSetting.txt";
GameSetting:addButton("Save Setting on next execution",function()
   SaveSetting()
end)
function LoadSetting()
   print("Loading Data")
   local HttpService = game:GetService("HttpService");
   if (readfile and isfile and isfile(filename)) then
      getgenv().SaveSetting = HttpService:JSONDecode(readfile(filename));
   end
end
function SaveSetting()
   print("Saving Data")
   local json;
   local HttpService = game:GetService("HttpService");
   if (writefile) then
      json = HttpService:JSONEncode(getgenv().SaveSetting);
      writefile(filename, json);
   else
      print("Can't save because of your trash exploits")
   end
end
wait()
local Games = lib:addPage("Game",6034227061)
local games = Games:addSection("Venyx is hot")
local PlaceID = game.PlaceId
local AllIDs = {}
local foundAnything = ""
local actualHour = os.date("!*t").hour
local Deleted = false
local File = pcall(function()
  AllIDs = game:GetService('HttpService'):JSONDecode(readfile("NotSameServers.json"))
end)

if not File then
  table.insert(AllIDs, actualHour)
  writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
end

function TPReturner()
   local Site;
   if foundAnything == "" then
      Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
   else
      Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
   end
   local ID = ""
   if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
      foundAnything = Site.nextPageCursor
   end
   local num = 0;
   for i,v in pairs(Site.data) do
      local Possible = true
      ID = tostring(v.id)
      if tonumber(v.maxPlayers) > tonumber(v.playing) then
         for _,Existing in pairs(AllIDs) do
            if num ~= 0 then
               if ID == tostring(Existing) then
                  Possible = false
               end
            else
               if tonumber(actualHour) ~= tonumber(Existing) then
                  local delFile = pcall(function()
                  delfile("NotSameServers.json")
                  AllIDs = {}
                  table.insert(AllIDs, actualHour)
                  end)
               end
            end
            num = num + 1
         end
         if Possible == true then
            table.insert(AllIDs, ID)
            wait()
            pcall(function()
               writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
               wait()
               game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
            end)
            wait(4)
         end
      end
   end
end

function Teleport()
   while game:GetService("RunService").RenderStepped:wait() do
      pcall(function()
         TPReturner()
         if foundAnything ~= "" then
            TPReturner()
         end
      end)
   end
end
games:addButton("Rejoin",function()
    local ts = game:GetService("TeleportService")
    local p = game:GetService("Players").LocalPlayer
    ts:Teleport(game.PlaceId, p)
 end)

 games:addButton("Teleport to lower server",function()

   getgenv().AutoTeleport = true
   getgenv().DontTeleportTheSameNumber = true 
   getgenv().CopytoClipboard = false
   
   
   if not game:IsLoaded() then
       print("Game is loading waiting... | Amnesia Empty Server Finder")
       end
   lib:Notify("Lower Server","Credit to Amnesia")
   local maxplayers = math.huge
   local serversmaxplayer;
   local goodserver;
   local gamelink = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
   
   function serversearch()
       for _, v in pairs(game:GetService("HttpService"):JSONDecode(game:HttpGetAsync(gamelink)).data) do
           if type(v) == "table" and v.playing ~= nil and maxplayers > v.playing then
               serversmaxplayer = v.maxPlayers
               maxplayers = v.playing
               goodserver = v.id
           end
       end
       print("Currently checking the servers with max this number of players : " .. maxplayers .. " | Amnesia Empty Server Finder")
   end
   
   function getservers()
       serversearch()
       for i,v in pairs(game:GetService("HttpService"):JSONDecode(game:HttpGetAsync(gamelink))) do
           if i == "nextPageCursor" then
               if gamelink:find("&cursor=") then
                   local a = gamelink:find("&cursor=")
                   local b = gamelink:sub(a)
                   gamelink = gamelink:gsub(b, "")
               end
               gamelink = gamelink .. "&cursor=" ..v
               getservers()
           end
       end
   end
   
   getservers()
   
       print("All of the servers are searched") 
      print("Server : " .. goodserver .. " Players : " .. maxplayers .. "/" .. serversmaxplayer .. " | Amnesia Empty Server Finder")
       if CopytoClipboard then
       setclipboard(goodserver)
       end
       if AutoTeleport then
           if DontTeleportTheSameNumber then 
               if #game:GetService("Players"):GetPlayers() - 4  == maxplayers then
                   return warn("It has same number of players (except you)")
               elseif goodserver == game.JobId then
                   return warn("Your current server is the most empty server atm") 
               end
           end
           print("AutoTeleport is enabled. Teleporting to : " .. goodserver)
           game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, goodserver)
       end
end)


games:addButton("Server Hop",function()
   Teleport()
end)

games:addButton("Copy Discord Server(Join or gay)",function()
   setclipboard("https://discord.gg/p2G3mXxfkA")
end)

games:addButton("Delete Ui",function()
   local ui = game:GetService("CoreGui"):FindFirstChild("Know Hub")
   if ui then
      ui:Destroy()
   end
end)

RTXmode = {
   "Autumn";
   "Summer";
}
games:addDropdown("RTX mode",RTXmode,function(Value)
   getgenv().mode = Value
end)

games:addButton("RTX Graphic",function()

   local a = game.Lighting
   a.TimeOfDay = 20
   a.Ambient = Color3.fromRGB(33, 33, 33)
   a.Brightness = 6.67
   a.ColorShift_Bottom = Color3.fromRGB(0, 0, 0)
   a.ColorShift_Top = Color3.fromRGB(255, 247, 237)
   a.EnvironmentDiffuseScale = 0.105
   a.EnvironmentSpecularScale = 0.522
   a.GlobalShadows = true
   a.OutdoorAmbient = Color3.fromRGB(51, 54, 67)
   a.ShadowSoftness = 0.04
   a.GeographicLatitude = -15.525
   a.ExposureCompensation = 0.75

   local b = Instance.new("BloomEffect", a)
   b.Enabled = true
   b.Intensity = 0.04
   b.Size = 1900
   b.Threshold = 0.915

   local c = Instance.new("ColorCorrectionEffect", a)
   c.Brightness = 0.176
   c.Contrast = 0.39
   c.Enabled = true
   c.Saturation = 0.2
   c.TintColor = Color3.fromRGB(217, 145, 57)
   if getgenv().mode == "Summer" then
      c.TintColor = Color3.fromRGB(255, 220, 148)
   elseif getgenv().mode == "Autumn" then
      c.TintColor = Color3.fromRGB(217, 145, 57)
   else
      lib:Notify("RTX mode","No mode selected!")
      lib:Notify("RTX mode","Please select a mode")
      b:Destroy()
      c:Destroy()
   end

   local d = Instance.new("DepthOfFieldEffect", a)
   d.Enabled = true
   d.FarIntensity = 0.077
   d.FocusDistance = 21.54
   d.InFocusRadius = 20.77
   d.NearIntensity = 0.277

   local e = Instance.new("ColorCorrectionEffect", a)
   e.Brightness = 0
   e.Contrast = -0.07
   e.Saturation = 0
   e.Enabled = true
   e.TintColor = Color3.fromRGB(255, 247, 239)

   local e2 = Instance.new("ColorCorrectionEffect", a)
   e2.Brightness = 0.2
   e2.Contrast = 0.45
   e2.Saturation = -0.1
   e2.Enabled = true
   e2.TintColor = Color3.fromRGB(255, 255, 255)

   local s = Instance.new("SunRaysEffect", a)
   s.Enabled = true
   s.Intensity = 0.01
   s.Spread = 0.146
end)

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
     if  getgenv().Queue then
      if getgenv().ObservationHop == true then queue_on_teleport([[ getgenv().ObservationHop = true getgenv().Observation = true ]])
      elseif getgenv().AutoCavanderHop == true then queue_on_teleport([[ getgenv().AutoCavanderHop = true]])
      elseif getgenv().AutoDoflamingoHop == true then queue_on_teleport([[ getgenv().AutoDoflamingoHop = true]])
      elseif getgenv().AutoEnelHop == true then queue_on_teleport([[ getgenv().AutoEnelHop = true]])
      elseif getgenv().AutoReaperHop == true then queue_on_teleport([[ getgenv().AutoReaperHop = true ]])
      elseif getgenv().AutoEliteHop == true then queue_on_teleport([[ getgenv().AutoEliteHop = true getgenv().AutoElites = true ]])
      elseif getgenv().AutoIndraHop == true then queue_on_teleport([[ getgenv().AutoIndraHop = true ]])
      elseif getgenv().AutoWaterKeyHop == true then queue_on_teleport([[ getgenv().AutoWaterKeyHop ]])
      elseif getgenv().AutoBlackBeardHop == true then queue_on_teleport([[ getgenv().AutoBlackBeardHop = true ]])
      elseif getgenv().AutoLibraryKeyHop == true then queue_on_teleport([[ getgenv().AutoLibraryKeyHop  = true]])
      elseif getgenv().AutoShankHop == true then queue_on_teleport([[ getgenv().AutoShankHop = true]])

      end
      wait(1)
      queue_on_teleport([[
         repeat wait(1)
         until game:IsLoaded()
         print("Queued")
         getgenv().Queue = true
         repeat wait(1)
         until game:GetService("Players").LocalPlayer.PlayerGui:FindFirstChild("LoadingScreen") == nil
        loadstring(game:HttpGet(('https://raw.githubusercontent.com/vinhuchi/Island_Game/main/Beta-Testing.lua'),true))()
         print("Loadstring loaded")

       print(getgenv().AutoCavanderHop)
      print(getgenv().AutoEnelHop)
      print(getgenv().AutoDoflamingoHop)
      print(getgenv().AutoReaperHop)
      ]])
      if getgenv().ObservationHop == true then
         wait(60)
         Teleport()
      else
         wait(150)
         Teleport()
      end
     end

   end
end)
AutoHopBoss = ""

games:addButton("Farm Ken Haki Hop",function()
   getgenv().Queue = true
   getgenv().Observation = true
   getgenv().ObservationHop = true
end)

games:addButton("Auto Cavander Hop",function()
   getgenv().AutoCavanderHop = true
   getgenv().Queue = true
end)
games:addButton("Auto Pole Hop",function()
   getgenv().AutoEnelHop = true
   getgenv().Queue = true
end)
games:addButton("Auto Swan Glasses Hop",function()
   getgenv().AutoDoflamingoHop = true
   getgenv().Queue = true
end)
games:addButton("Auto Indra Hop",function()
   getgenv().AutoIndraHop = true
   getgenv().Queue = true
end)

games:addButton("Auto BlackBeard Hop",function()
   getgenv().AutoBlackBeardHop = true
   getgenv().Queue = true
end)
games:addButton("Auto Water Key Hop",function()
   getgenv().AutoWaterKeyHop = true
   getgenv().Queue = true
end)
games:addButton("Auto BlackBeard Hop",function()
   getgenv().AutoLibraryKeyHop= true
   getgenv().Queue = true
end)
games:addButton("Auto Saber Hop",function()
   getgenv().AutoShankHop = true
   getgenv().Queue = true
   
end)

games:addButton("Auto Elite Hop",function()
   getgenv().AutoEliteHop = true
   getgenv().AutoElites = true
   getgenv().Queue = true
   --Teleport()
end)
games:addButton("Auto Hallow Scythe Hop",function()
   getgenv().AutoReaperHop = true
   getgenv().Queue = true
end)
spawn(function()
   repeat wait(1)
   until game:GetService("Players").LocalPlayer.PlayerGui:FindFirstChild("LoadingScreen") == nil
   while wait() do
      if getgenv().AutoCavanderHop then
         if ThirdWorld then
            AutoHopBoss = "Beautiful Pirate [Lv. 1950] [Boss]"
            StringFind = "Beautiful Pirate"
            wait(1)
            local name = tostring(game.Players.LocalPlayer)
            local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
            for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
                  v.Function()
            end
            AutoHopBossValue = true

            wait()
            lib:Notify("Auto Cavander Hop","Auto Cavander Hop Ran")
         else
            lib:Notify("Auto Cavander Hop","You must go to Third Sea")
            getgenv().AutoCavanderHop = false
         end
      elseif getgenv().AutoDoflamingoHop then
         if SecondWorld then
            AutoHopBoss = "Don Swan [Lv. 1000] [Boss]"
            StringFind = "Don Swan"
            print(AutoHopBoss)
            print(StringFind)
            AutoHopBossValue = true
            wait(1)
            local name = tostring(game.Players.LocalPlayer)
            local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
            for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
               v.Function()
            end

            wait()
            lib:Notify("Auto Swan Glasses Hop","Auto Doflamingo Hop Ran")

         else
            lib:Notify("Auto Swan Glasses Hop","You must go to Second Sea")
            getgenv().AutoDoflamingoHop = false
         end
      elseif getgenv().AutoReaperHop then
         if ThirdWorld then
            AutoHopBoss = "Soul Reaper [Lv. 2100] [Raid Boss]"
            StringFind = "Soul Reaper"
            wait(1)
            local name = tostring(game.Players.LocalPlayer)
            local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
            for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
                  v.Function()
            end
            AutoHopBossValue = true

            wait()
            lib:Notify("Auto Hallow Scythe Hop","Auto Reaper Hop Ran")
         else
            lib:Notify("Auto Hallow Scythe Hop","You must go to Third Sea")
            getgenv().AutoReaperHop = false
         end
      elseif getgenv().AutoEnelHop then
         if FirstWorld then
            AutoHopBoss = "Thunder God [Lv. 575] [Boss]"
            StringFind = "Thunder"
            print(StringFind)
            print(AutoHopBoss)
            AutoHopBossValue = true
            wait(1)
            local name = tostring(game.Players.LocalPlayer)
            local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
            for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
               v.Function()
            end

            lib:Notify("Auto Pole Hop","Auto Enel Hop Ran")
         else
            lib:Notify("Auto Pole Hop","You must go to First Sea")
            getgenv().AutoEnelHop = false
         end
      elseif getgenv().AutoShankHop then
         if FirstWorld then
            AutoHopBoss = "Saber Expert [Lv. 200] [Boss]"
            StringFind = "Saber"
            print(StringFind)
            print(AutoHopBoss)
            AutoHopBossValue = true
            wait(1)
            local name = tostring(game.Players.LocalPlayer)
            local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
            for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
               v.Function()
            end

            lib:Notify("Auto Saber Hop","Auto Shank Hop Ran")
         else
            lib:Notify("Auto Saber Hop","You must go to First Sea")
            getgenv().AutoShankHop = false
         end
      elseif getgenv().AutoBlackBeardHop then
         if SecondWorld then
            AutoHopBoss = "Darkbeard [Lv. 1000] [Raid Boss]"
            StringFind = "Darkbeard"
            print(StringFind)
            print(AutoHopBoss)
            AutoHopBossValue = true
            wait(1)
            local name = tostring(game.Players.LocalPlayer)
            local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
            for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
               v.Function()
            end

            lib:Notify("Auto Black Beard Hop","Auto Black Beard Ran")
         else
            lib:Notify("Auto Black Beard Hop","You must go to First Sea")
            getgenv().AutoBlackBeardHop  = false
         end
   elseif getgenv().AutoLibraryKeyHop then
      if SecondWorld then
         AutoHopBoss = "Awakened Ice Admiral [Lv. 1400] [Boss]"
         StringFind = "Awakened Ice"
         print(StringFind)
         print(AutoHopBoss)
         AutoHopBossValue = true
         wait(1)
         local name = tostring(game.Players.LocalPlayer)
         local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
         for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
            v.Function()
         end
      
         lib:Notify("Auto Library Key Hop","Auto Awakened Ice Ran")
      else
         lib:Notify("Auto Library Key Hop","You must go to First Sea")
         getgenv().AutoLibraryKeyHop = false
      end
   elseif getgenv().AutoWaterKeyHop then
      if SecondWorld then
         AutoHopBoss = "Tide Keeper [Lv. 1475] [Boss]"
         StringFind = "Tide Keeper"
         print(StringFind)
         print(AutoHopBoss)
         AutoHopBossValue = true
         wait(1)
         local name = tostring(game.Players.LocalPlayer)
         local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
         for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
            v.Function()
         end

         lib:Notify("Auto Water Key Hop","Auto Tide Keeper  Ran")
      else
         lib:Notify("Auto Water Key Hop","You must go to First Sea")
         getgenv().AutoWaterKeyHop  = false
      end

   elseif getgenv().AutoIndraHop then
      if ThirdWorld then
         AutoHopBoss = "rip_indra True Form [Lv. 5000] [Raid Boss]"
         StringFind = "rip_indra"
         print(StringFind)
         print(AutoHopBoss)
         AutoHopBossValue = true
         wait(1)
         local name = tostring(game.Players.LocalPlayer)
         local MyUiButton = game.Players.LocalPlayer.PlayerGui.Main.ChooseTeam.Container.Marines.Frame.ViewportFrame.TextButton
         for i,v in pairs(getconnections(MyUiButton.MouseButton1Click)) do
            v.Function()
         end

         lib:Notify("Auto Indra Hop","Auto Indra Ran")
      else
         lib:Notify("Auto Indra Hop","You must go to Third Sea")
         getgenv().AutoWaterKeyHop  = false
      end
   end
      wait(1)
      if getgenv().AutoDoflamingoHop or getgenv().AutoCavanderHop or getgenv().AutoEnelHop or getgenv().AutoReaperHop  then
         
         if AutoHopBoss == nil or StringFind == nil then wait(1) else
         wait(1)
         if game:GetService("Workspace").Enemies:FindFirstChild(AutoHopBoss) and AutoHopBossValue == true then
            CoolDown = false
            for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
               if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and string.find(v.Name, StringFind) then
                        TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeed)
                        repeat game:GetService("RunService").RenderStepped:wait()
                           if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 300 and not Tweening then
                           pcall(function()
                              local RigC = require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework)
                              RigC.activeController.attacking = false 
                              RigC.activeController.active = false
                              RigC.activeController.timeToNextAttack = 0
                           end)
                           for i,v in pairs(ListMelee) do
                              if game.Players.LocalPlayer.Backpack:FindFirstChild(ListMelee[i]) then
                                 local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ListMelee[i])
                                 game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
                              end
                           end
                        randomnumber = math.random(1,4)
                        if randomnumber == 1  then
                           wait()
                           game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                           elseif randomnumber ==2   then
                              wait()
                              game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                           elseif randomnumber ==3  then
                              wait()
                              game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                           elseif randomnumber ==4  then
                              wait()
                              game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                           end
                           VirtualUser:CaptureController()
                           VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
                           zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
                           zeroGrav(v.HumanoidRootPart)                         
                           v.Humanoid.PlatformStand = true
                           game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 15, 15)
                           v.HumanoidRootPart.Transparency = 0.9
                           v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                           v.HumanoidRootPart.CanCollide = false
                           v.HumanoidRootPart.Color = Color3.fromRGB(0, 0, 0)
                           end
                        until v.Humanoid.Health <= 0
                           if v.Humanoid.Health <= 0 then
                              local args = {[1] = "SetTeam",[2] = "Marines"}
                              game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
                              wait(1)
                              Teleport()
                           end
                  wait(1)
                  CoolDown = true
               end
            end
         else print("No", StringFind) wait(1) Teleport()
         end
      end
      end
   end
end)


spawn(function()
   LoadSetting()
end)

spawn(function()
   while wait() do
      if game.Players.LocalPlayer.Character.Humanoid.Health <= 0 or game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") == nil then
         
         wait(5)
         TeleportTween(game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame,TweenSpeed)
         print("Stopped")
         wait(1)
      end
   end
end)
function Moveenemies()
   for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
      if v:IsA("Model") and v.Name:find("1100") and v.Name ~= "Lab Subordinate [Lv. 1100]" then
         v.Parent = game:GetService("Workspace").Boats
      end
   end
end
Mode = "Magnet"
function autofarm() -- AutoFarmm Function
   LocalPlayer = game.Players.LocalPlayer
   if LocalPlayer.PlayerGui.Main.Quest.Visible == false then
      MagnetOn = false
      CheckQuest()
      Tweening = true
      TeleportTween(CFrameQuest,TweenSpeed) 
      wait(0.5)
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuest, LevelQuest)
      TeleportTween(CFrameMon,TweenSpeedF) 
      Tweening = false
      wait(0.5)
      if AutoSetSpawn then 
         local args = {
            [1] = "SetSpawnPoint"
         }

         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      end
   elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible == true and AF then
      CheckQuest()
      -- Moveenemies()
      -- LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
      if game:GetService("Workspace").Enemies:FindFirstChild(Ms) then   
         pcall(function() -- no magnet
            for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
               if v.Name ==  Ms then
                  if sethiddenproperty then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                  end
                  if setsimulationradius then
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  v.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0,30,0)
                  repeat wait()
                     if game:GetService("Workspace").Enemies:FindFirstChild(Ms) then  
                        if string.find(LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text, NameMon) then
                           wait(.1)
                           if sethiddenproperty then 
                              sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                           end
                           if setsimulationradius then
                              sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                           end
                           v.HumanoidRootPart.Transparency = 1
                           v.HumanoidRootPart.Size = Vector3.new(60, 60, 60) -- hitbox                                            
                           Hit = true
                           -- magnet
                           if Mode == "Magnet" then                    
                              if sethiddenproperty then 
                                 sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                              end
                              if setsimulationradius then
                                 sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                              end
                              local randomnumber = math.random(1, 4)
                              if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if randomnumber == 1 and AF then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                    elseif randomnumber ==2 and AF  then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                    elseif randomnumber ==3 and AF then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                    elseif randomnumber ==4 and AF then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                    end
                                    wait(0.4)
                                 end
                              elseif not Tweening then Tweening = true TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0),TweenSpeedF)  Tweening = false  
                                 if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                    if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                       if randomnumber == 1 and AF then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                       elseif randomnumber ==2 and AF  then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                       elseif randomnumber ==3 and AF then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                       elseif randomnumber ==4 and AF then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                       end
                                       wait(0.4)
                                    end
                                 end
                              end
                                 PosMon = v.HumanoidRootPart.CFrame
                                 PosMon2 = v.HumanoidRootPart.Position
                              spawn(function()
                                 repeat 
                                    wait(0.1)
                                    local randomnumber2 = math.random(1, 4)
                                    yMon = v.HumanoidRootPart.Position.Y
                                    if game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y < yMon then
                                       if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                          if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                             if randomnumber2 ==1  and AF  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                             elseif randomnumber2 ==2  and AF  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                             elseif randomnumber2 == 3  and AF  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance)
                                             elseif randomnumber2 == 4  and AF  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)
                                             end
                                          end
                                       elseif not Tweening then Tweening = true TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0),TweenSpeedF) Tweening = false 
                                          if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                             if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                                if randomnumber2 == 1  and AF  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                                elseif randomnumber2 ==2  and AF  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                                elseif randomnumber2 ==3  and AF  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                                elseif randomnumber2 ==4  and AF  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                                end
                                             end
                                          end
                                       end
                                    end
                                 until v.Humanoid.Health <= 0 or not AF
                              end)

                              spawn(function()
                                 wait(5)
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                                       v.HumanoidRootPart.CFrame = PosMon
                                    end
                                 end
                                 CurrentHealth = v.Humanoid.Health
                                 
                                 wait(5)
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if CurrentHealth == v.Humanoid.Health or v.Humanoid.Health == v.Humanoid.MaxHealth then
                                       if v and AF then
                                          if sethiddenproperty then 
                                             sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                                          end
                                          if setsimulationradius then
                                             sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                                          end
                                          if v.Humanoid.Health > 0 then
                                             v.Humanoid.Health = 0 
                                          elseif v.Humanoid.Health == 0 then
                                             v.Humanoid.Health = v.Humanoid.MaxHealth
                                          else
                                             v.Humanoid.Health = 0 
                                          end
                                          v:breakJoints()
                                       end
                                    end
                                 end
                                 wait(5)
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if v and AF then
                                       if sethiddenproperty then 
                                          sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                                       end
                                       if setsimulationradius then
                                          sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                                       end
                                       if v.Humanoid.Health > 0 then
                                          v.Humanoid.Health = 0 
                                       elseif v.Humanoid.Health == 0 then
                                          v.Humanoid.Health = v.Humanoid.MaxHealth
                                       else
                                          v.Humanoid.Health = 0 
                                       end
                                       v:breakJoints()
                                    end
                                 end
                              end)
 
                              MagnetOn = true
                           else
                              local randomnumber = math.random(1, 4)
                              if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening  then
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if randomnumber == 1 and AF then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                    elseif randomnumber ==2 and AF  then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                    elseif randomnumber ==3 and AF then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                    elseif randomnumber ==4 and AF then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                    end     
                                    wait(0.4)
                                 end
                              elseif not Tweening then wait(1)
                                 if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                    if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v.Humanoid.Health > 0 then
                                       if randomnumber == 1 and AF then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                       elseif randomnumber ==2 and AF  then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                       elseif randomnumber ==3 and AF then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                       elseif randomnumber ==4 and AF then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                       end
                                       wait(0.4)
                                    end
                                 end
                              end
                           end
                           v.HumanoidRootPart.CanCollide = false
                        else
                           MagnetOn = false
                           CheckQuest()
                           -- print()
                           -- LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameQuest
                           wait(0.5)
                           game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuest, LevelQuest)
                           end
                        else
                           -- CheckQuest() 
                           -- game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
                     end
                  until not v.Parent or v.Humanoid.Health <= 0 or AF == false or LocalPlayer.PlayerGui.Main.Quest.Visible == false
                  Hit = false
                  -- CheckQuest()
                  -- game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
               end
            end
         end)
      else
             -- CheckQuest()
             -- game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
      end 
   end
end

function AutoFarmBoss()
	CheckQuestBoss()
	if SelectBoss == "Don Swan [Lv. 1000] [Boss]" or SelectBoss == "Cursed Captain [Lv. 1325] [Raid Boss]" or SelectBoss == "Saber Expert [Lv. 200] [Boss]" or SelectBoss == "Mob Leader [Lv. 120] [Boss]" or SelectBoss == "Darkbeard [Lv. 1000] [Raid Boss]" then
		if game:GetService("Workspace").Enemies:FindFirstChild(SelectBoss) then
			for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
				if FarmBoss and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and v.Name == MsBoss then

               TeleportTween(v.HumanoidRootPart.CFrame,TweenSpeed)
					repeat
                  wait() 
                  if FarmBoss and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 300 then
						   pcall(function() 
                        if HideHitBlox then
                           v.HumanoidRootPart.Transparency = 1
                        else
                           v.HumanoidRootPart.Transparency = 0.75
                        end
                        v.HumanoidRootPart.CanCollide = false
                        v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                        game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 17, 5)
                        zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
                        VirtualUser:CaptureController()
                        VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
						   end)
                  end
					until not FarmBoss or not v.Parent or v.Humanoid.Health <= 0
				end
			end
		else
			TeleportTween(CFrameBoss,TweenSpeed)
		end
	elseif SelectBoss == "Order [Lv. 1250] [Raid Boss]" then
		if game:GetService("Workspace").Enemies:FindFirstChild(SelectBoss) then
			for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
				if FarmBoss and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and v.Name == MsBoss then
               TeleportTween(v.HumanoidRootPart.CFrame)
					repeat
                  wait() 
						pcall(function() 
							if HideHitBlox then
								v.HumanoidRootPart.Transparency = 1
							else
								v.HumanoidRootPart.Transparency = 0.75
							end
							v.HumanoidRootPart.CanCollide = false
							v.HumanoidRootPart.Size = Vector3.new(80, 80, 80)
							game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 25, 25)
                     zeroGrav(game.Players.LocalPlayer.Character.HumanoidRootPart)
   						VirtualUser:CaptureController()
							VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
						end)
					until not FarmBoss or not v.Parent or v.Humanoid.Health <= 0
				end
			end
		else
			TeleportTween(CFrameBoss,TweenSpeed)
		end
	else
		if game:GetService("Workspace").Enemies:FindFirstChild(SelectBoss) or game:GetService("ReplicatedStorage"):FindFirstChild(SelectBoss) then
			if game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == false then
            print("How Crashed1")
				CheckQuestBoss()
				TeleportTween(CFrameQuestBoss,TweenSpeed)
				wait(1.5)
				game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuestBoss, LevelQuestBoss)
				wait(1)
            TeleportTween(CFrameBoss,TweenSpeed)
            print("Tweened")
			elseif game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == true then
				for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
					if FarmBoss and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and v.Name == MsBoss then
                  TeleportTween(CFrameBoss,TweenSpeed)
                  print("Tweened")
						repeat
                     wait() 
							pcall(function() 
								if HideHitBlox then
									v.HumanoidRootPart.Transparency = 1
								else
									v.HumanoidRootPart.Transparency = 0.75
								end
								v.HumanoidRootPart.CanCollide = false
								v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
								game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 17, 5)
								VirtualUser:CaptureController()
								VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
							end)
						until not FarmBoss or not v.Parent or v.Humanoid.Health <= 0 or game.Players.LocalPlayer.PlayerGui.Main.Quest.Visible == false
					end
				end
            print("How Crashed3")
				TeleportTween(CFrameBoss,TweenSpeed)
			end
		end
	end
end

function autofarmm() -- AutoFarmm Function
   LocalPlayer = game.Players.LocalPlayer
   if LocalPlayer.PlayerGui.Main.Quest.Visible == false then
      MagnetOn = false
      CheckQuest2()
      if CFrameQuestm == nil then
         lib:Notify("AutoFarm","Broken or didn't select a mob")
         return false
      end
      Tweening = true
      TeleportTween(CFrameQuestm,TweenSpeed) 
      wait(0.5)
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuestm, LevelQuestm)
      TeleportTween(CFrameMonm,TweenSpeedF) 
      Tweening = false
      wait(0.5)
   elseif game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible == true and AFM then
      CheckQuest2()
      -- Moveenemies()
      -- LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
      if game:GetService("Workspace").Enemies:FindFirstChild(MobsPick) then   
         pcall(function() -- no magnet
            for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
               if v.Name ==  MobsPick then
                  if sethiddenproperty then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                  end
                  if setsimulationradius then
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  v.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0,30,0)
                  repeat wait()
                     if game:GetService("Workspace").Enemies:FindFirstChild(MobsPick) then  
                        if string.find(LocalPlayer.PlayerGui.Main.Quest.Container.QuestTitle.Title.Text, NameMonm) then
                           wait(.1)
                           if sethiddenproperty then 
                              sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                           end
                           if setsimulationradius then
                              sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                           end
                           v.HumanoidRootPart.Transparency = 1
                           v.HumanoidRootPart.Size = Vector3.new(60, 60, 60) -- hitbox                                            
                           Hit = true
                           -- magnet
                           if Mode == "Magnet" then                    
                              if sethiddenproperty then 
                                 sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                              end
                              if setsimulationradius then
                                 sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                              end
                              local randomnumber = math.random(1, 4)
                              if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if randomnumber == 1 and AFM then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                    elseif randomnumber ==2 and AFM  then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                    elseif randomnumber ==3 and AFM then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                    elseif randomnumber ==4 and AFM then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                           
                                    end
                                    wait(0.4)
                                 end
                              elseif not Tweening then Tweening = true TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0),TweenSpeedF)  Tweening = false  
                                 if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                    if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                       if randomnumber == 1 and AFM then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                       elseif randomnumber ==2 and AFM  then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                       elseif randomnumber ==3 and AFM then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                       elseif randomnumber ==4 and AFM then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                          
                                       end
                                       wait(0.4)
                                    end
                                 end
                              end
                              if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                 PosMonm = v.HumanoidRootPart.CFrame
                                 PosMonm2 = v.HumanoidRootPart.Position
                              end
                              spawn(function()
                                 repeat 
                                    wait(0.1)
                                    local randomnumber2 = math.random(1, 4)
                                    yMon = v.HumanoidRootPart.Position.Y
                                    if game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y < yMon then
                                       if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                          if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                             if randomnumber2 ==1  and AFM  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                             elseif randomnumber2 ==2  and AFM  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                             elseif randomnumber2 == 3  and AFM  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance)
                                             elseif randomnumber2 == 4  and AFM  then
                                                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)
                                             end
                                          end
                                       elseif not Tweening then wait(1)
                                          if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                             if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                                if randomnumber2 == 1  and AFM  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                                elseif randomnumber2 ==2  and AFM  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                                elseif randomnumber2 ==3  and AFM  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                                elseif randomnumber2 ==4  and AFM  then
                                                   game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                                end
                                             end
                                          end
                                       end
                                    end
                                 until v.Humanoid.Health <= 0 or not AFM
                              end)

                              spawn(function()
                                 wait(5)
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                                       v.HumanoidRootPart.CFrame = PosMonm
                                    end
                                 end
                                 CurrentHealth = v.Humanoid.Health
                                 
                                 wait(5)
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if CurrentHealth == v.Humanoid.Health or v.Humanoid.Health == v.Humanoid.MaxHealth then
                                       if v and AFM then
                                          if sethiddenproperty then 
                                             sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                                          end
                                          if setsimulationradius then
                                             sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                                          end
                                          if v.Humanoid.Health > 0 then
                                             v.Humanoid.Health = 0 
                                          elseif v.Humanoid.Health == 0 then
                                             v.Humanoid.Health = v.Humanoid.MaxHealth
                                          else
                                             v.Humanoid.Health = 0 
                                          end
                                          v:breakJoints()
                                       end
                                    end
                                 end
                                 wait(5)
                                 if v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if v and AFM then
                                       if sethiddenproperty then 
                                          sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                                       end
                                       if setsimulationradius then
                                          sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                                       end
                                       if v.Humanoid.Health > 0 then
                                          v.Humanoid.Health = 0 
                                       elseif v.Humanoid.Health == 0 then
                                          v.Humanoid.Health = v.Humanoid.MaxHealth
                                       else
                                          v.Humanoid.Health = 0 
                                       end
                                       v:breakJoints()
                                    end
                                 end
                              end)
 
                              MagnetOn = true
                           else
                              local randomnumber = math.random(1, 4)
                              if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening  then
                                 if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") then
                                    if randomnumber == 1 and AFM then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                       
                                    elseif randomnumber ==2 and AFM  then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                    elseif randomnumber ==3 and AFM then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                    elseif randomnumber ==4 and AFM then
                                       game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                    end     
                                    wait(0.4)
                                 end
                              elseif not Tweening then Tweening = true TeleportTween(v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0),TweenSpeedF)  Tweening = false  
                                 if (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 350 and not Tweening then
                                    if game.Players.LocalPlayer.Character.Humanoid.Health > 0 and v.Humanoid.Health > 0 then
                                       if randomnumber == 1 and AFM then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().Distance)      
                                       elseif randomnumber ==2 and AFM  then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 0, -getgenv().Distance) 
                                       elseif randomnumber ==3 and AFM then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().Distance, 0)
                                       elseif randomnumber ==4 and AFM then
                                          game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, -getgenv().Distance, 0)
                                       end
                                       wait(0.4)
                                    end
                                 end
                              end
                           end
                           v.HumanoidRootPart.CanCollide = false
                        else
                           MagnetOn = false
                           CheckQuest2()
                           -- print()
                           -- LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameQuest
                           wait(0.5)
                           game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", NaemQuestm, LevelQuestm)
                           end
                        else
                           -- CheckQuest() 
                           -- game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
                     end
                  until not v.Parent or v.Humanoid.Health <= 0 or AFM == false or LocalPlayer.PlayerGui.Main.Quest.Visible == false
                  Hit = false
                  -- CheckQuest()
                  -- game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
               end
            end
         end)
      else
             -- CheckQuest()
             -- game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrameMon
      end 
   end
end
-- MagnetMode AFM
spawn(function() 
   -- local testmagnet = require(game:GetService("ReplicatedStorage").Quests)
   while game:GetService("RunService").RenderStepped:wait() do
      if Mode == "Magnet" and AFM and MagnetOn and not Tweening then
         CheckQuest2()

         pcall(function()
            repeat
               Moveenemies()
               --  testmagnet.BartiloQuest[1].OnComplete(Ms,PosMon)
               game:GetService("RunService").RenderStepped:wait()
               for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                  if sethiddenproperty then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                  end
                  if setsimulationradius then
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  if v.Name ~= "Ice Admiral [Lv. 700] [Boss]" and v.Name ~= "Saber Expert [Lv. 200] [Boss]" and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance and not Tweening and v.Name == MobsPick and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") ~= nil then
                     zeroGrav(v.HumanoidRootPart)
                     v.Humanoid.PlatformStand = true
                     v.HumanoidRootPart.Transparency = 1
                     v.HumanoidRootPart.CanCollide = false
                     v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                     if (v.HumanoidRootPart.Position - PosMonm2).magnitude <= 350 then
                        if sethiddenproperty then 
                           sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                        end
                        if setsimulationradius then
                           sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                        end
                        v.HumanoidRootPart.CFrame = PosMonm * CFrame.new(0,0,0)
                     else
                        PosMonm = v.HumanoidRootPart.CFrame 
                        PosMonm2 = v.HumanoidRootPart.Position
                     end
                     v.Humanoid.WalkSpeed = 8
                  end
               end
            until LocalPlayer.PlayerGui.Main.Quest.Visible == false or AFM == false or MagnetOn == false
         end)
      end
   end
end)
-- MagnetMode AF
spawn(function() 
   -- local testmagnet = require(game:GetService("ReplicatedStorage").Quests)
   while game:GetService("RunService").RenderStepped:wait() do
      if Mode == "Magnet" and AF and MagnetOn and not Tweening then
         CheckQuest()

         pcall(function()
            repeat
               Moveenemies()
               --  testmagnet.BartiloQuest[1].OnComplete(Ms,PosMon)
               game:GetService("RunService").RenderStepped:wait()
               for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                  if sethiddenproperty then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                  end
                  if setsimulationradius then
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  if v.Name ~= "Ice Admiral [Lv. 700] [Boss]" and v.Name ~= "Saber Expert [Lv. 200] [Boss]" and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance and not Tweening and v.Name == Ms and v.Humanoid.Health > 0 and v.Parent and v:FindFirstChild("HumanoidRootPart") ~= nil then
                     zeroGrav(v.HumanoidRootPart)
                     v.Humanoid.PlatformStand = true
                     v.HumanoidRootPart.Transparency = 1
                     v.HumanoidRootPart.CanCollide = false
                     if sethiddenproperty then 
                        sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                     end
                     if setsimulationradius then
                        sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                     end
                     v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                     if (v.HumanoidRootPart.Position - PosMon2).magnitude <= 350 then
                        v.HumanoidRootPart.CFrame = PosMon * CFrame.new(0,0,0)
                     else
                        PosMon = v.HumanoidRootPart.CFrame 
                        PosMon2 = v.HumanoidRootPart.Position
                     end
                     v.Humanoid.WalkSpeed = 8
                  end
               end
            until LocalPlayer.PlayerGui.Main.Quest.Visible == false or AF == false or MagnetOn == false
         end)
      end
   end
end)
 spawn(function() -- MagnetMode MobAura
   -- local testmagnet = require(game:GetService("ReplicatedStorage").Quests)
    while game:GetService("RunService").RenderStepped:wait() do
         if MobAura and getgenv().MobAuraBring then

               pcall(function()
                     repeat
                        Moveenemies()
                             --  testmagnet.BartiloQuest[1].OnComplete(Ms,PosMon)
                        game:GetService("RunService").RenderStepped:wait()
                        for i, v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                           if v.Name ~= "Ice Admiral [Lv. 700] [Boss]" and v.Name ~= "Saber Expert [Lv. 200] [Boss]" and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= getgenv().MagnetDistance then
                              zeroGrav(v.HumanoidRootPart)
                              v.Humanoid.PlatformStand = true
                              v.HumanoidRootPart.Transparency = 1
                              v.HumanoidRootPart.CanCollide = false
                              v.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                              v.HumanoidRootPart.CFrame = MobAuraBringPos * CFrame.new(0,0,0)
                              v.Humanoid.WalkSpeed = 8

                            end
                        end
                     until MobAura == false or getgenv().MobAuraBring == false
               end)
         end
   end
end)

 spawn(function()
   local testkillaura = require(game:GetService("ReplicatedStorage").Quests)
   while game:GetService("RunService").RenderStepped:wait(0.5) do
      if killaura then
         pcall(function()
            -- repeat
            -- game:GetService("RunService").RenderStepped:wait()
            for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
               if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 1000 then
                  if sethiddenproperty then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                  end
                  if setsimulationradius then
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  if v.Humanoid.Health > 0 then
                     v.Humanoid.Health = 0 
                  elseif v.Humanoid.Health == 0 then
                     v.Humanoid.Health = v.Humanoid.MaxHealth
                  else
                     v.Humanoid.Health = 0 
                  end
                  -- v.HumanoidRootPart.Anchored = true
                  -- v.HumanoidRootPart.CFrame = game:service'Players'.LocalPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(0, -25, 0)
                  v.HumanoidRootPart.Transparency = 0.9
                  if sethiddenproperty then 
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius",  10000)
                  end
                  if setsimulationradius then
                     sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                  end
                  v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                  v.HumanoidRootPart.CanCollide = false
                  v.HumanoidRootPart.Color = Color3.new(255, 0, 0)
                  v.Humanoid:ChangeState(15)
                  wait(0.1)
               end
            end
             -- until killaura == false
          end)
       end
    end
 end)

--Put all Npc into Enemies Folder

 
 
 -- spawn(function()
 --    while wait(0.1) do
 --      if killaura then
 --        pcall(function()
 --             for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
 --                if v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v:IsA("Model") and 
 --                (v.HumanoidRootPart.Position-game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position).magnitude <= 1000 then
 --                --    repeat wait(0.1)
 --                      Namekill = v
 --                      Poskill = v.v.HumanoidRootPart.CFrame
 --                      setsimulationradius(math.huge)
 --                      -- v.HumanoidRootPart.CFrame = game:service'Players'.LocalPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(0, -25, 0)
 --                      v.HumanoidRootPart.Transparency = 0.9
 --                      v.Humanoid:ChangeState(15)
 --                      v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
 --                      v.HumanoidRootPart.CanCollide = false
 --                      v.HumanoidRootPart.Color = Color3.new(255, 0, 0)
 --                --    until v.Humanoid.Health <= 0
 --                end
 --            end
 --        end)
 --      end
 --    end
 -- end)
 
spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      if AF then
         autofarm()
      end
      if AFM then
         autofarmm()
      end
   end
end)

spawn(function()
   while game:GetService("RunService").RenderStepped:wait() do
      if (FarmBoss and game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Quest.Visible == true) or (AFM and game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Quest.Visible == true) or (AF and game:GetService("Players")["LocalPlayer"].PlayerGui.Main.Quest.Visible == true) then
         EquipWeapon(SelectToolWeapon)
      end
   end
end)
 
local NoCDATK = require(game.Players.LocalPlayer.PlayerScripts.CombatFramework)

function Click()
   if AFM == true or AF == true or BartiloQuest == true or AutoSecworld == true or AutoThirdworld == true or Allboss == true or MobAura == true 
   or pk == true or getgenv().AutoElites == true or RainbowHaki == true then
      if getgenv().FastAttack then
         if NoCDATK.activeController and game.Players.LocalPlayer.Character.Humanoid.Health > 0 then
            if NoCDATK.activeController.timeToNextAttack and NoCDATK.activeController.timeToNextAttack ~= 0 then 
               NoCDATK.activeController.timeToNextAttack =  0
            end
         end
      end
   end
end

game.Players.LocalPlayer:GetMouse().Button1Down:Connect(Click)

         

spawn(function() -- Hit
   while game:GetService("RunService").RenderStepped:wait(0.2) do
   if getgenv().AutoFarmClick then
      if Hit then
            VirtualUser:CaptureController()
            VirtualUser:ClickButton1(Vector2.new(851, 158), game:GetService("Workspace").Camera.CFrame)
         end
      end
   end
end)
if SecondWorld then
   function BartiloQuest() -- Barilo
      Mobx = "Swan Pirate [Lv. 775]"
      QuestName = "BartiloQuest"
      MobName = "Swan Pirate"
      QuestLevel = 1
      CFrameQuestx = CFrame.new(-461.533203, 72.3478546, 300.311096, 0.050853312, -0, -0.998706102, 0, 1, -0, 0.998706102, 0, 0.050853312)
   end
end
-- Saber Quest
if FirstWorld then
   function autosaber() 

      if AF == true then
         AF = false
         ChangeAF = true
      end
      if AFM == true then
         AFM = false
         ChangeAFM = true
      end

      local min = 10
      local args = {
         [1] = "ProQuestProgress",
         [2] = "Plate",
         [3] = 1
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "Plate",
         [3] = 2
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "Plate",
         [3] = 3
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "Plate",
         [3] = 4
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "Plate",
         [3] = 5
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "GetTorch"
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "DestroyTorch"
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "GetCup"
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
   
      local toolname = "Cup"
      local Plr = game:GetService("Players").LocalPlayer
      for i = 1,1 do
         pcall(function()
            if Plr.Backpack:FindFirstChild(toolname) and Plr.Character:FindFirstChild(toolname) == nil then
               local tool = Plr.Backpack:FindFirstChild(toolname)
               Plr.Character.Humanoid:EquipTool(tool)
            end
         end)
      end

      local args = {
         [1] = "ProQuestProgress",
         [2] = "FillCup",
         [3] = game:GetService("Players").LocalPlayer.Character.Cup
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "SickMan"
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

      local args = {
         [1] = "ProQuestProgress",
         [2] = "RichSon"
      }
      game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
      print("Fill Cup !")
      wait(1)
   
      game:service'Players'.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-2845.2168, 7.73550653, 5333.45313) -- Mob Leader
      print("Tp to Mob !")
   
      wait(3)
      print("Hit Leader !")
      if game.Workspace.Enemies:FindFirstChild("Mob Leader [Lv. 120] [Boss]")then
         CheckMob = true
         Hit = true
         for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
            if CheckMob and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 and v.Name == "Mob Leader [Lv. 120] [Boss]" then
               repeat wait(.1)
                  EquipWeapon(SelectToolWeapon)
                  pcall(function()
                     v.HumanoidRootPart.Transparency = 0.5
                     v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                     v.HumanoidRootPart.BrickColor = BrickColor.new("White")
                     v.HumanoidRootPart.CanCollide = false
                     game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame*CFrame.new(0, 20, 10)
                  end)
               until not CheckMob or not v.Parent or v.Humanoid.Health <= 0 or ASaber == false
            end
         end
         CheckMob = false
         Hit = false
         wait(1)
         print("Quest Relic!")

         local args = {
            [1] = "ProQuestProgress",
            [2] = "RichSon"
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

         wait(2)
         local args = {
            [1] = "ProQuestProgress"
         }
         game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
   
   
         local toolname = "Relic"
         local Plr = game:GetService("Players").LocalPlayer
         for i = 1,1 do
            pcall(function()
               if Plr.Backpack:FindFirstChild(toolname) and Plr.Character:FindFirstChild(toolname) == nil then
                  local tool = Plr.Backpack:FindFirstChild(toolname)
                  Plr.Character.Humanoid:EquipTool(tool)
               end
            end)
         end
   
         local vDoor = game:GetService("Workspace").Map.Jungle.Final.Invis
         vDoor.CanCollide = false
         vDoor.Size = Vector3.new(20,20,20)
   
         wait(1)
         TeleportTween(CFrame.new(-1403.92944, 29.8519993, 6.61151266),TweenSpeedF)
   
         wait(3)
         print("Hit Shank ")
         if game.Workspace.Enemies:FindFirstChild("Saber Expert [Lv. 200] [Boss]")then
            CheckSaber = true
            Hit = true
            for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
               if CheckSaber and v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart")and v.Name == "Saber Expert [Lv. 200] [Boss]" then
                  repeat wait(.1)
                     if game.Players.LocalPlayer.Character:FindFirstChild("SelectToolWeapon") == nil then
                        EquipWeapon(SelectToolWeapon)
                     end
                     pcall(function()
                        v.HumanoidRootPart.Transparency = 0.5
                        v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                        v.HumanoidRootPart.BrickColor = BrickColor.new("White")
                        v.HumanoidRootPart.CanCollide = false
                        TeleportTween(v.HumanoidRootPart.CFrame*CFrame.new(0, 20, 10),TweenSpeedF)
                     end)
                  until not CheckSaber or not v.Parent or v.Humanoid.Health <= 0 or ASaber == false
               end
            end
            CheckSaber = false
            Hit = false
   
            wait(0.5)
            local args = {
               [1] = "KenTalk",
               [2] = "Buy"
            }
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))

            ASaber = false
            wait(5)

            if ChangeAF == true then
               AF = true
               ChangeAF = false
            end
            if ChangeAFM == true then
               AFM = true
               ChangeAFM = false
            end

            wait(60 * min)
         end
      end
   end
end
lib:Notify("Know Hub","Loaded")
