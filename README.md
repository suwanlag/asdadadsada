-- SETTING
NAME_OF_HUB = "DEHED-HUB"

-- UI

local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

local input = game:GetService("UserInputService")
local run = game:GetService("RunService")
local tween = game:GetService("TweenService")
local tweeninfo = TweenInfo.new

local utility = {}

local objects = {}
local themes = {
    Background = Color3.fromRGB(24, 24, 24),
    Glow = Color3.fromRGB(0, 0, 0),
    Accent = Color3.fromRGB(10, 10, 10),
    LightContrast = Color3.fromRGB(20, 20, 20),
    DarkContrast = Color3.fromRGB(14, 14, 14),
    TextColor = Color3.fromRGB(0, 255, 255)
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
        for i, v in pairs(table) do
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

        spawn(
            function()
                wait(0.2)

                object.ImageTransparency = 0
                clone:Destroy()
            end
        )

        return clone
    end

    function utility:InitializeKeybind()
        self.keybinds = {}
        self.ended = {}

        input.InputBegan:Connect(
            function(key, proc)
                if self.keybinds[key.KeyCode] and not proc then
                    for i, bind in pairs(self.keybinds[key.KeyCode]) do
                        bind()
                    end
                end
            end
        )

        input.InputEnded:Connect(
            function(key)
                if key.UserInputType == Enum.UserInputType.MouseButton1 then
                    for i, callback in pairs(self.ended) do
                        callback()
                    end
                end
            end
        )
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

        while key.UserInputType ~= Enum.UserInputType.Keyboard do
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

        frame.InputBegan:Connect(
            function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = true
                    mousePos = input.Position
                    framePos = parent.Position

                    input.Changed:Connect(
                        function()
                            if input.UserInputState == Enum.UserInputState.End then
                                dragging = false
                            end
                        end
                    )
                end
            end
        )

        frame.InputChanged:Connect(
            function(input)
                if input.UserInputType == Enum.UserInputType.MouseMovement then
                    dragInput = input
                end
            end
        )

        input.InputChanged:Connect(
            function(input)
                if input == dragInput and dragging then
                    local delta = input.Position - mousePos
                    parent.Position =
                        UDim2.new(
                        framePos.X.Scale,
                        framePos.X.Offset + delta.X,
                        framePos.Y.Scale,
                        framePos.Y.Offset + delta.Y
                    )
                end
            end
        )
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
        local container =
            utility:Create(
            "ScreenGui",
            {
                Name = title,
                Parent = game.CoreGui
            },
            {
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Main",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0.25, 0, 0.052435593, 0),
                        Size = UDim2.new(0, 511, 0, 428),
                        Image = "rbxassetid://4641149554",
                        ImageColor3 = themes.Background,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(4, 4, 296, 296)
                    },
                    {
                        utility:Create(
                            "ImageLabel",
                            {
                                Name = "Glow",
                                BackgroundTransparency = 1,
                                Position = UDim2.new(0, -15, 0, -15),
                                Size = UDim2.new(1, 30, 1, 30),
                                ZIndex = 0,
                                Image = "rbxassetid://5028857084",
                                ImageColor3 = themes.Glow,
                                ScaleType = Enum.ScaleType.Slice,
                                SliceCenter = Rect.new(24, 24, 276, 276)
                            }
                        ),
                        utility:Create(
                            "ImageLabel",
                            {
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
                            },
                            {
                                utility:Create(
                                    "ScrollingFrame",
                                    {
                                        Name = "Pages_Container",
                                        Active = true,
                                        BackgroundTransparency = 1,
                                        Position = UDim2.new(0, 0, 0, 10),
                                        Size = UDim2.new(1, 0, 1, -20),
                                        CanvasSize = UDim2.new(0, 0, 0, 314),
                                        ScrollBarThickness = 0
                                    },
                                    {
                                        utility:Create(
                                            "UIListLayout",
                                            {
                                                SortOrder = Enum.SortOrder.LayoutOrder,
                                                Padding = UDim.new(0, 10)
                                            }
                                        )
                                    }
                                )
                            }
                        ),
                        utility:Create(
                            "ImageLabel",
                            {
                                Name = "TopBar",
                                BackgroundTransparency = 1,
                                ClipsDescendants = true,
                                Size = UDim2.new(1, 0, 0, 38),
                                ZIndex = 5,
                                Image = "rbxassetid://4595286933",
                                ImageColor3 = themes.Accent,
                                ScaleType = Enum.ScaleType.Slice,
                                SliceCenter = Rect.new(4, 4, 296, 296)
                            },
                            {
                                utility:Create(
                                    "TextLabel",
                                    {
                                        -- title
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
                                    }
                                )
                            }
                        )
                    }
                )
            }
        )

        utility:InitializeKeybind()
        utility:DraggingEnabled(container.Main.TopBar, container.Main)

        return setmetatable(
            {
                container = container,
                pagesContainer = container.Main.Pages.Pages_Container,
                pages = {}
            },
            library
        )
    end

    function page.new(library, title, icon)
        local button =
            utility:Create(
            "TextButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                icon and
                    utility:Create(
                        "ImageLabel",
                        {
                            Name = "Icon",
                            AnchorPoint = Vector2.new(0, 0.5),
                            BackgroundTransparency = 1,
                            Position = UDim2.new(0, 12, 0.5, 0),
                            Size = UDim2.new(0, 16, 0, 16),
                            ZIndex = 3,
                            Image = "rbxassetid://" .. tostring(icon),
                            ImageColor3 = themes.TextColor,
                            ImageTransparency = 0.64
                        }
                    ) or
                    {}
            }
        )

        local container =
            utility:Create(
            "ScrollingFrame",
            {
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
            },
            {
                utility:Create(
                    "UIListLayout",
                    {
                        SortOrder = Enum.SortOrder.LayoutOrder,
                        Padding = UDim.new(0, 10)
                    }
                )
            }
        )

        return setmetatable(
            {
                library = library,
                container = container,
                button = button,
                sections = {}
            },
            page
        )
    end

    function section.new(page, title)
        local container =
            utility:Create(
            "ImageLabel",
            {
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
            },
            {
                utility:Create(
                    "Frame",
                    {
                        Name = "Container",
                        Active = true,
                        BackgroundTransparency = 1,
                        BorderSizePixel = 0,
                        Position = UDim2.new(0, 8, 0, 8),
                        Size = UDim2.new(1, -16, 1, -16)
                    },
                    {
                        utility:Create(
                            "TextLabel",
                            {
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
                            }
                        ),
                        utility:Create(
                            "UIListLayout",
                            {
                                SortOrder = Enum.SortOrder.LayoutOrder,
                                Padding = UDim.new(0, 4)
                            }
                        )
                    }
                )
            }
        )

        return setmetatable(
            {
                page = page,
                container = container.Container,
                colorpickers = {},
                modules = {},
                binds = {},
                lists = {}
            },
            section
        )
    end

    function library:addPage(...)
        local page = page.new(self, ...)
        local button = page.button

        table.insert(self.pages, page)

        button.MouseButton1Click:Connect(
            function()
                self:SelectPage(page, true)
            end
        )

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
            utility:Tween(
                container,
                {
                    Size = UDim2.new(0, 511, 0, 428),
                    Position = self.position
                },
                0.2
            )
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

            utility:Tween(
                container,
                {
                    Size = UDim2.new(0, 511, 0, 0),
                    Position = self.position + UDim2.new(0, 0, 0, 428)
                },
                0.2
            )
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
        local notification =
            utility:Create(
            "ImageLabel",
            {
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
            },
            {
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Flash",
                        Size = UDim2.new(1, 0, 1, 0),
                        BackgroundTransparency = 1,
                        Image = "rbxassetid://4641149554",
                        ImageColor3 = themes.TextColor,
                        ZIndex = 5
                    }
                ),
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Glow",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, -15, 0, -15),
                        Size = UDim2.new(1, 30, 1, 30),
                        ZIndex = 2,
                        Image = "rbxassetid://5028857084",
                        ImageColor3 = themes.Glow,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(24, 24, 276, 276)
                    }
                ),
                utility:Create(
                    "TextLabel",
                    {
                        Name = "Title",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, 10, 0, 8),
                        Size = UDim2.new(1, -40, 0, 16),
                        ZIndex = 4,
                        Font = Enum.Font.GothamSemibold,
                        TextColor3 = themes.TextColor,
                        TextSize = 14.000,
                        TextXAlignment = Enum.TextXAlignment.Left
                    }
                ),
                utility:Create(
                    "TextLabel",
                    {
                        Name = "Text",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, 10, 1, -24),
                        Size = UDim2.new(1, -40, 0, 16),
                        ZIndex = 4,
                        Font = Enum.Font.Gotham,
                        TextColor3 = themes.TextColor,
                        TextSize = 12.000,
                        TextXAlignment = Enum.TextXAlignment.Left
                    }
                ),
                utility:Create(
                    "ImageButton",
                    {
                        Name = "Accept",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(1, -26, 0, 8),
                        Size = UDim2.new(0, 16, 0, 16),
                        Image = "rbxassetid://5012538259",
                        ImageColor3 = themes.TextColor,
                        ZIndex = 4
                    }
                ),
                utility:Create(
                    "ImageButton",
                    {
                        Name = "Decline",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(1, -26, 1, -24),
                        Size = UDim2.new(0, 16, 0, 16),
                        Image = "rbxassetid://5012538583",
                        ImageColor3 = themes.TextColor,
                        ZIndex = 4
                    }
                )
            }
        )

        -- dragging
        utility:DraggingEnabled(notification)

        -- position and size
        title = title or "Notification"
        text = text or ""

        notification.Title.Text = title
        notification.Text.Text = text

        local padding = 10
        local textSize =
            game:GetService("TextService"):GetTextSize(text, 12, Enum.Font.Gotham, Vector2.new(math.huge, 16))

        notification.Position =
            library.lastNotification or UDim2.new(0, padding, 1, -(notification.AbsoluteSize.Y + padding))
        notification.Size = UDim2.new(0, 0, 0, 60)

        utility:Tween(notification, {Size = UDim2.new(0, textSize.X + 70, 0, 60)}, 0.2)
        wait(0.2)

        notification.ClipsDescendants = false
        utility:Tween(
            notification.Flash,
            {
                Size = UDim2.new(0, 0, 0, 60),
                Position = UDim2.new(1, 0, 0, 0)
            },
            0.2
        )

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
            utility:Tween(
                notification,
                {
                    Size = UDim2.new(0, 0, 0, 60),
                    Position = notification.Position + UDim2.new(0, textSize.X + 70, 0, 0)
                },
                0.2
            )

            wait(0.2)
            notification:Destroy()
        end

        self.activeNotification = close

        notification.Accept.MouseButton1Click:Connect(
            function()
                if not active then
                    return
                end

                if callback then
                    callback(true)
                end

                close()
            end
        )

        notification.Decline.MouseButton1Click:Connect(
            function()
                if not active then
                    return
                end

                if callback then
                    callback(false)
                end

                close()
            end
        )
    end

    function section:addButton(title, callback)
        local button =
            utility:Create(
            "ImageButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
                        Name = "Title",
                        BackgroundTransparency = 1,
                        Size = UDim2.new(1, 0, 1, 0),
                        ZIndex = 3,
                        Font = Enum.Font.Gotham,
                        Text = title,
                        TextColor3 = themes.TextColor,
                        TextSize = 12,
                        TextTransparency = 0.10000000149012
                    }
                )
            }
        )

        table.insert(self.modules, button)
        --self:Resize()

        local text = button.Title
        local debounce

        button.MouseButton1Click:Connect(
            function()
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
                    callback(
                        function(...)
                            self:updateButton(button, ...)
                        end
                    )
                end

                debounce = false
            end
        )

        return button
    end

    function section:addToggle(title, default, callback)
        local toggle =
            utility:Create(
            "ImageButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                utility:Create(
                    "ImageLabel",
                    {
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
                    },
                    {
                        utility:Create(
                            "ImageLabel",
                            {
                                Name = "Frame",
                                BackgroundTransparency = 1,
                                Position = UDim2.new(0, 2, 0.5, -6),
                                Size = UDim2.new(1, -22, 1, -4),
                                ZIndex = 2,
                                Image = "rbxassetid://5028857472",
                                ImageColor3 = themes.TextColor,
                                ScaleType = Enum.ScaleType.Slice,
                                SliceCenter = Rect.new(2, 2, 298, 298)
                            }
                        )
                    }
                )
            }
        )

        table.insert(self.modules, toggle)
        --self:Resize()

        local active = default
        self:updateToggle(toggle, nil, active)

        toggle.MouseButton1Click:Connect(
            function()
                active = not active
                self:updateToggle(toggle, nil, active)

                if callback then
                    callback(
                        active,
                        function(...)
                            self:updateToggle(toggle, ...)
                        end
                    )
                end
            end
        )

        return toggle
    end

    function section:addTextbox(title, default, callback)
        local textbox =
            utility:Create(
            "ImageButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Button",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(1, -110, 0.5, -8),
                        Size = UDim2.new(0, 100, 0, 16),
                        ZIndex = 2,
                        Image = "rbxassetid://5028857472",
                        ImageColor3 = themes.LightContrast,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(2, 2, 298, 298)
                    },
                    {
                        utility:Create(
                            "TextBox",
                            {
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
                            }
                        )
                    }
                )
            }
        )

        table.insert(self.modules, textbox)
        --self:Resize()

        local button = textbox.Button
        local input = button.Textbox

        textbox.MouseButton1Click:Connect(
            function()
                if textbox.Button.Size ~= UDim2.new(0, 100, 0, 16) then
                    return
                end

                utility:Tween(
                    textbox.Button,
                    {
                        Size = UDim2.new(0, 200, 0, 16),
                        Position = UDim2.new(1, -210, 0.5, -8)
                    },
                    0.2
                )

                wait()

                input.TextXAlignment = Enum.TextXAlignment.Left
                input:CaptureFocus()
            end
        )

        input:GetPropertyChangedSignal("Text"):Connect(
            function()
                if
                    button.ImageTransparency == 0 and
                        (button.Size == UDim2.new(0, 200, 0, 16) or button.Size == UDim2.new(0, 100, 0, 16))
                 then -- i know, i dont like this either
                    utility:Pop(button, 10)
                end

                if callback then
                    callback(
                        input.Text,
                        nil,
                        function(...)
                            self:updateTextbox(textbox, ...)
                        end
                    )
                end
            end
        )

        input.FocusLost:Connect(
            function()
                input.TextXAlignment = Enum.TextXAlignment.Center

                utility:Tween(
                    textbox.Button,
                    {
                        Size = UDim2.new(0, 100, 0, 16),
                        Position = UDim2.new(1, -110, 0.5, -8)
                    },
                    0.2
                )

                if callback then
                    callback(
                        input.Text,
                        true,
                        function(...)
                            self:updateTextbox(textbox, ...)
                        end
                    )
                end
            end
        )

        return textbox
    end

    function section:addKeybind(title, default, callback, changedCallback)
        local keybind =
            utility:Create(
            "ImageButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Button",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(1, -110, 0.5, -8),
                        Size = UDim2.new(0, 100, 0, 16),
                        ZIndex = 2,
                        Image = "rbxassetid://5028857472",
                        ImageColor3 = themes.LightContrast,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(2, 2, 298, 298)
                    },
                    {
                        utility:Create(
                            "TextLabel",
                            {
                                Name = "Text",
                                BackgroundTransparency = 1,
                                ClipsDescendants = true,
                                Size = UDim2.new(1, 0, 1, 0),
                                ZIndex = 3,
                                Font = Enum.Font.GothamSemibold,
                                Text = default and default.Name or "None",
                                TextColor3 = themes.TextColor,
                                TextSize = 11
                            }
                        )
                    }
                )
            }
        )

        table.insert(self.modules, keybind)
        --self:Resize()

        local text = keybind.Button.Text
        local button = keybind.Button

        local animate = function()
            if button.ImageTransparency == 0 then
                utility:Pop(button, 10)
            end
        end

        self.binds[keybind] = {
            callback = function()
                animate()

                if callback then
                    callback(
                        function(...)
                            self:updateKeybind(keybind, ...)
                        end
                    )
                end
            end
        }

        if default and callback then
            self:updateKeybind(keybind, nil, default)
        end

        keybind.MouseButton1Click:Connect(
            function()
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
                        changedCallback(
                            key,
                            function(...)
                                self:updateKeybind(keybind, ...)
                            end
                        )
                    end
                end
            end
        )

        return keybind
    end

    function section:addColorPicker(title, default, callback)
        local colorpicker =
            utility:Create(
            "ImageButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                utility:Create(
                    "ImageButton",
                    {
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
                    }
                )
            }
        )

        local tab =
            utility:Create(
            "ImageLabel",
            {
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
                Visible = false
            },
            {
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Glow",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, -15, 0, -15),
                        Size = UDim2.new(1, 30, 1, 30),
                        ZIndex = 0,
                        Image = "rbxassetid://5028857084",
                        ImageColor3 = themes.Glow,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(22, 22, 278, 278)
                    }
                ),
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                utility:Create(
                    "ImageButton",
                    {
                        Name = "Close",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(1, -26, 0, 8),
                        Size = UDim2.new(0, 16, 0, 16),
                        ZIndex = 2,
                        Image = "rbxassetid://5012538583",
                        ImageColor3 = themes.TextColor
                    }
                ),
                utility:Create(
                    "Frame",
                    {
                        Name = "Container",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, 8, 0, 32),
                        Size = UDim2.new(1, -18, 1, -40)
                    },
                    {
                        utility:Create(
                            "UIListLayout",
                            {
                                SortOrder = Enum.SortOrder.LayoutOrder,
                                Padding = UDim.new(0, 6)
                            }
                        ),
                        utility:Create(
                            "ImageButton",
                            {
                                Name = "Canvas",
                                BackgroundTransparency = 1,
                                BorderColor3 = themes.LightContrast,
                                Size = UDim2.new(1, 0, 0, 60),
                                AutoButtonColor = false,
                                Image = "rbxassetid://5108535320",
                                ImageColor3 = Color3.fromRGB(255, 0, 0),
                                ScaleType = Enum.ScaleType.Slice,
                                SliceCenter = Rect.new(2, 2, 298, 298)
                            },
                            {
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "White_Overlay",
                                        BackgroundTransparency = 1,
                                        Size = UDim2.new(1, 0, 0, 60),
                                        Image = "rbxassetid://5107152351",
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    }
                                ),
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "Black_Overlay",
                                        BackgroundTransparency = 1,
                                        Size = UDim2.new(1, 0, 0, 60),
                                        Image = "rbxassetid://5107152095",
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    }
                                ),
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "Cursor",
                                        BackgroundColor3 = themes.TextColor,
                                        AnchorPoint = Vector2.new(0.5, 0.5),
                                        BackgroundTransparency = 1.000,
                                        Size = UDim2.new(0, 10, 0, 10),
                                        Position = UDim2.new(0, 0, 0, 0),
                                        Image = "rbxassetid://5100115962",
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    }
                                )
                            }
                        ),
                        utility:Create(
                            "ImageButton",
                            {
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
                            },
                            {
                                utility:Create(
                                    "Frame",
                                    {
                                        Name = "Select",
                                        BackgroundColor3 = themes.TextColor,
                                        BorderSizePixel = 1,
                                        Position = UDim2.new(1, 0, 0, 0),
                                        Size = UDim2.new(0, 2, 1, 0),
                                        ZIndex = 2
                                    }
                                ),
                                utility:Create(
                                    "UIGradient",
                                    {
                                        -- rainbow canvas
                                        Color = ColorSequence.new(
                                            {
                                                ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 0, 0)),
                                                ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 255, 0)),
                                                ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 255, 0)),
                                                ColorSequenceKeypoint.new(0.50, Color3.fromRGB(0, 255, 255)),
                                                ColorSequenceKeypoint.new(0.66, Color3.fromRGB(0, 0, 255)),
                                                ColorSequenceKeypoint.new(0.82, Color3.fromRGB(255, 0, 255)),
                                                ColorSequenceKeypoint.new(1.00, Color3.fromRGB(255, 0, 0))
                                            }
                                        )
                                    }
                                )
                            }
                        ),
                        utility:Create(
                            "Frame",
                            {
                                Name = "Inputs",
                                BackgroundTransparency = 1,
                                Position = UDim2.new(0, 10, 0, 158),
                                Size = UDim2.new(1, 0, 0, 16)
                            },
                            {
                                utility:Create(
                                    "UIListLayout",
                                    {
                                        FillDirection = Enum.FillDirection.Horizontal,
                                        SortOrder = Enum.SortOrder.LayoutOrder,
                                        Padding = UDim.new(0, 6)
                                    }
                                ),
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "R",
                                        BackgroundTransparency = 1,
                                        BorderSizePixel = 0,
                                        Size = UDim2.new(0.305, 0, 1, 0),
                                        ZIndex = 2,
                                        Image = "rbxassetid://5028857472",
                                        ImageColor3 = themes.DarkContrast,
                                        ScaleType = Enum.ScaleType.Slice,
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    },
                                    {
                                        utility:Create(
                                            "TextLabel",
                                            {
                                                Name = "Text",
                                                BackgroundTransparency = 1,
                                                Size = UDim2.new(0.400000006, 0, 1, 0),
                                                ZIndex = 2,
                                                Font = Enum.Font.Gotham,
                                                Text = "R:",
                                                TextColor3 = themes.TextColor,
                                                TextSize = 10.000
                                            }
                                        ),
                                        utility:Create(
                                            "TextBox",
                                            {
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
                                            }
                                        )
                                    }
                                ),
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "G",
                                        BackgroundTransparency = 1,
                                        BorderSizePixel = 0,
                                        Size = UDim2.new(0.305, 0, 1, 0),
                                        ZIndex = 2,
                                        Image = "rbxassetid://5028857472",
                                        ImageColor3 = themes.DarkContrast,
                                        ScaleType = Enum.ScaleType.Slice,
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    },
                                    {
                                        utility:Create(
                                            "TextLabel",
                                            {
                                                Name = "Text",
                                                BackgroundTransparency = 1,
                                                ZIndex = 2,
                                                Size = UDim2.new(0.400000006, 0, 1, 0),
                                                Font = Enum.Font.Gotham,
                                                Text = "G:",
                                                TextColor3 = themes.TextColor,
                                                TextSize = 10.000
                                            }
                                        ),
                                        utility:Create(
                                            "TextBox",
                                            {
                                                Name = "Textbox",
                                                BackgroundTransparency = 1,
                                                Position = UDim2.new(0.300000012, 0, 0, 0),
                                                Size = UDim2.new(0.600000024, 0, 1, 0),
                                                ZIndex = 2,
                                                Font = Enum.Font.Gotham,
                                                Text = "255",
                                                TextColor3 = themes.TextColor,
                                                TextSize = 10.000
                                            }
                                        )
                                    }
                                ),
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "B",
                                        BackgroundTransparency = 1,
                                        BorderSizePixel = 0,
                                        Size = UDim2.new(0.305, 0, 1, 0),
                                        ZIndex = 2,
                                        Image = "rbxassetid://5028857472",
                                        ImageColor3 = themes.DarkContrast,
                                        ScaleType = Enum.ScaleType.Slice,
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    },
                                    {
                                        utility:Create(
                                            "TextLabel",
                                            {
                                                Name = "Text",
                                                BackgroundTransparency = 1,
                                                Size = UDim2.new(0.400000006, 0, 1, 0),
                                                ZIndex = 2,
                                                Font = Enum.Font.Gotham,
                                                Text = "B:",
                                                TextColor3 = themes.TextColor,
                                                TextSize = 10.000
                                            }
                                        ),
                                        utility:Create(
                                            "TextBox",
                                            {
                                                Name = "Textbox",
                                                BackgroundTransparency = 1,
                                                Position = UDim2.new(0.300000012, 0, 0, 0),
                                                Size = UDim2.new(0.600000024, 0, 1, 0),
                                                ZIndex = 2,
                                                Font = Enum.Font.Gotham,
                                                Text = "255",
                                                TextColor3 = themes.TextColor,
                                                TextSize = 10.000
                                            }
                                        )
                                    }
                                )
                            }
                        ),
                        utility:Create(
                            "ImageButton",
                            {
                                Name = "Button",
                                BackgroundTransparency = 1,
                                BorderSizePixel = 0,
                                Size = UDim2.new(1, 0, 0, 20),
                                ZIndex = 2,
                                Image = "rbxassetid://5028857472",
                                ImageColor3 = themes.DarkContrast,
                                ScaleType = Enum.ScaleType.Slice,
                                SliceCenter = Rect.new(2, 2, 298, 298)
                            },
                            {
                                utility:Create(
                                    "TextLabel",
                                    {
                                        Name = "Text",
                                        BackgroundTransparency = 1,
                                        Size = UDim2.new(1, 0, 1, 0),
                                        ZIndex = 3,
                                        Font = Enum.Font.Gotham,
                                        Text = "Submit",
                                        TextColor3 = themes.TextColor,
                                        TextSize = 11.000
                                    }
                                )
                            }
                        )
                    }
                )
            }
        )

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
                callback(
                    value,
                    function(...)
                        self:updateColorPicker(colorpicker, ...)
                    end
                )
            end
        end

        utility:DraggingEnded(
            function()
                draggingColor, draggingCanvas = false, false
            end
        )

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

                textbox.Focused:Connect(
                    function()
                        focused = true
                    end
                )

                textbox.FocusLost:Connect(
                    function()
                        focused = false

                        if not tonumber(textbox.Text) then
                            textbox.Text = math.floor(rgb[container.Name:lower()])
                        end
                    end
                )

                textbox:GetPropertyChangedSignal("Text"):Connect(
                    function()
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
                    end
                )
            end
        end

        canvas.MouseButton1Down:Connect(
            function()
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
            end
        )

        color.MouseButton1Down:Connect(
            function()
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
            end
        )

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

                local x1, x2 = button.AbsoluteSize.X / 2, 162
                 --tab.AbsoluteSize.X
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

        tab.Container.Button.MouseButton1Click:Connect(
            function()
                animate()
            end
        )

        tab.Close.MouseButton1Click:Connect(
            function()
                self:updateColorPicker(colorpicker, nil, lastColor)
                animate()
            end
        )

        return colorpicker
    end

    function section:addSlider(title, default, min, max, callback)
        local slider =
            utility:Create(
            "ImageButton",
            {
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
            },
            {
                utility:Create(
                    "TextLabel",
                    {
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
                    }
                ),
                utility:Create(
                    "TextBox",
                    {
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
                    }
                ),
                utility:Create(
                    "TextLabel",
                    {
                        Name = "Slider",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, 10, 0, 28),
                        Size = UDim2.new(1, -20, 0, 16),
                        ZIndex = 3,
                        Text = ""
                    },
                    {
                        utility:Create(
                            "ImageLabel",
                            {
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
                            },
                            {
                                utility:Create(
                                    "ImageLabel",
                                    {
                                        Name = "Fill",
                                        BackgroundTransparency = 1,
                                        Size = UDim2.new(0.8, 0, 1, 0),
                                        ZIndex = 3,
                                        Image = "rbxassetid://5028857472",
                                        ImageColor3 = themes.TextColor,
                                        ScaleType = Enum.ScaleType.Slice,
                                        SliceCenter = Rect.new(2, 2, 298, 298)
                                    },
                                    {
                                        utility:Create(
                                            "ImageLabel",
                                            {
                                                Name = "Circle",
                                                AnchorPoint = Vector2.new(0.5, 0.5),
                                                BackgroundTransparency = 1,
                                                ImageTransparency = 1.000,
                                                ImageColor3 = themes.TextColor,
                                                Position = UDim2.new(1, 0, 0.5, 0),
                                                Size = UDim2.new(0, 10, 0, 10),
                                                ZIndex = 3,
                                                Image = "rbxassetid://4608020054"
                                            }
                                        )
                                    }
                                )
                            }
                        )
                    }
                )
            }
        )

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
                callback(
                    value,
                    function(...)
                        self:updateSlider(slider, ...)
                    end
                )
            end
        end

        self:updateSlider(slider, nil, value, min, max)

        utility:DraggingEnded(
            function()
                dragging = false
            end
        )

        slider.MouseButton1Down:Connect(
            function(input)
                dragging = true

                while dragging do
                    utility:Tween(circle, {ImageTransparency = 0}, 0.1)

                    value = self:updateSlider(slider, nil, nil, min, max, value)
                    callback(value)

                    utility:Wait()
                end

                wait(0.5)
                utility:Tween(circle, {ImageTransparency = 1}, 0.2)
            end
        )

        textbox.FocusLost:Connect(
            function()
                if not tonumber(textbox.Text) then
                    value = self:updateSlider(slider, nil, default or min, min, max)
                    callback(value)
                end
            end
        )

        textbox:GetPropertyChangedSignal("Text"):Connect(
            function()
                local text = textbox.Text

                if not allowed[text] and not tonumber(text) then
                    textbox.Text = text:sub(1, #text - 1)
                elseif not allowed[text] then
                    value = self:updateSlider(slider, nil, tonumber(text) or value, min, max)
                    callback(value)
                end
            end
        )

        return slider
    end

    function section:addDropdown(title, list, callback)
        local dropdown =
            utility:Create(
            "Frame",
            {
                Name = "Dropdown",
                Parent = self.container,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 30),
                ClipsDescendants = true
            },
            {
                utility:Create(
                    "UIListLayout",
                    {
                        SortOrder = Enum.SortOrder.LayoutOrder,
                        Padding = UDim.new(0, 4)
                    }
                ),
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "Search",
                        BackgroundTransparency = 1,
                        BorderSizePixel = 0,
                        Size = UDim2.new(1, 0, 0, 30),
                        ZIndex = 2,
                        Image = "rbxassetid://5028857472",
                        ImageColor3 = themes.DarkContrast,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(2, 2, 298, 298)
                    },
                    {
                        utility:Create(
                            "TextBox",
                            {
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
                            }
                        ),
                        utility:Create(
                            "ImageButton",
                            {
                                Name = "Button",
                                BackgroundTransparency = 1,
                                BorderSizePixel = 0,
                                Position = UDim2.new(1, -28, 0.5, -9),
                                Size = UDim2.new(0, 18, 0, 18),
                                ZIndex = 3,
                                Image = "rbxassetid://5012539403",
                                ImageColor3 = themes.TextColor,
                                SliceCenter = Rect.new(2, 2, 298, 298)
                            }
                        )
                    }
                ),
                utility:Create(
                    "ImageLabel",
                    {
                        Name = "List",
                        BackgroundTransparency = 1,
                        BorderSizePixel = 0,
                        Size = UDim2.new(1, 0, 1, -34),
                        ZIndex = 2,
                        Image = "rbxassetid://5028857472",
                        ImageColor3 = themes.Background,
                        ScaleType = Enum.ScaleType.Slice,
                        SliceCenter = Rect.new(2, 2, 298, 298)
                    },
                    {
                        utility:Create(
                            "ScrollingFrame",
                            {
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
                            },
                            {
                                utility:Create(
                                    "UIListLayout",
                                    {
                                        SortOrder = Enum.SortOrder.LayoutOrder,
                                        Padding = UDim.new(0, 4)
                                    }
                                )
                            }
                        )
                    }
                )
            }
        )

        table.insert(self.modules, dropdown)
        --self:Resize()

        local search = dropdown.Search
        local focused

        list = list or {}

        search.Button.MouseButton1Click:Connect(
            function()
                if search.Button.Rotation == 0 then
                    self:updateDropdown(dropdown, nil, list, callback)
                else
                    self:updateDropdown(dropdown, nil, nil, callback)
                end
            end
        )

        search.TextBox.Focused:Connect(
            function()
                if search.Button.Rotation == 0 then
                    self:updateDropdown(dropdown, nil, list, callback)
                end

                focused = true
            end
        )

        search.TextBox.FocusLost:Connect(
            function()
                focused = false
            end
        )

        search.TextBox:GetPropertyChangedSignal("Text"):Connect(
            function()
                if focused then
                    local list = utility:Sort(search.TextBox.Text, list)
                    list = #list ~= 0 and list

                    self:updateDropdown(dropdown, nil, list, callback)
                end
            end
        )

        dropdown:GetPropertyChangedSignal("Size"):Connect(
            function()
                self:Resize()
            end
        )

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

        error("No module found under " .. tostring(info))
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

        utility:Tween(
            frame,
            {
                Size = UDim2.new(1, -22, 1, -9),
                Position = position[value] + UDim2.new(0, 0, 0, 2.5)
            },
            0.2
        )

        wait(0.1)
        utility:Tween(
            frame,
            {
                Size = UDim2.new(1, -22, 1, -4),
                Position = position[value]
            },
            0.1
        )
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
            local button =
                utility:Create(
                "ImageButton",
                {
                    Parent = dropdown.List.Frame,
                    BackgroundTransparency = 1,
                    BorderSizePixel = 0,
                    Size = UDim2.new(1, 0, 0, 30),
                    ZIndex = 2,
                    Image = "rbxassetid://5028857472",
                    ImageColor3 = themes.DarkContrast,
                    ScaleType = Enum.ScaleType.Slice,
                    SliceCenter = Rect.new(2, 2, 298, 298)
                },
                {
                    utility:Create(
                        "TextLabel",
                        {
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
                        }
                    )
                }
            )

            button.MouseButton1Click:Connect(
                function()
                    if callback then
                        callback(
                            value,
                            function(...)
                                self:updateDropdown(dropdown, ...)
                            end
                        )
                    end

                    self:updateDropdown(dropdown, value, nil, callback)
                end
            )

            entries = entries + 1
        end

        local frame = dropdown.List.Frame

        utility:Tween(
            dropdown,
            {Size = UDim2.new(1, 0, 0, (entries == 0 and 30) or math.clamp(entries, 0, 3) * 34 + 38)},
            0.3
        )
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

-- SCRIPT

function autohaki()
    pcall(
        function()
            if game.Players.LocalPlayer.Character:FindFirstChild("HasBuso") then
            else
                local args = {
                    [1] = "Buso"
                }

                game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
            end
        end
    )
end

function EquipWeapon(ToolSe)
    if game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe) then
        local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe)
        wait(.4)
        game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
    end
end

local function Attack()
    game:GetService "VirtualUser":CaptureController()
    game:GetService "VirtualUser":Button1Down(Vector2.new(1280, 672))
end

local function AbandonQ()
    local args = {
        [1] = "AbandonQuest"
    }
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end

local function GetQuest(s, x)
    local args = {
        [1] = "StartQuest",
        [2] = s,
        [3] = x
    }
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
end

local function Teleport(p)
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = p
end

local function CheckLevel()
    local LEVEL = game:GetService("Players").LocalPlayer.Data.Level.Value
    if game.PlaceId == 2753915549 then
        if LEVEL == 1 or LEVEL <= 9 then
            MONNAME = "Bandit [Lv. 5]"
            QUESTNAME = "BanditQuest1"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                1059.16223,
                16.3627396,
                1548.61841,
                -0.937358618,
                9.94775604e-08,
                0.348365903,
                9.82571038e-08,
                1,
                -2.11714699e-08,
                -0.348365903,
                1.43841641e-08,
                -0.937358618
            )
            PUK =
                CFrame.new(
                1095.95618,
                67.8474503,
                1617.60278,
                0.334988832,
                4.35448122e-08,
                -0.942222118,
                -4.86487686e-08,
                1,
                2.89188904e-08,
                0.942222118,
                3.61504391e-08,
                0.334988832
            )
            QUESTTEXT = "Reward:\n$350\n250 Exp."
        elseif LEVEL == 10 or LEVEL <= 14 then
            MONNAME = "Monkey [Lv. 14]"
            QUESTNAME = "JungleQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -1599.82532,
                36.8521347,
                153.959076,
                0.00275422214,
                5.1952032e-08,
                -0.999996185,
                1.33985356e-08,
                1,
                5.1989133e-08,
                0.999996185,
                -1.35416744e-08,
                0.00275422214
            )
            PUK = CFrame.new(-1610.2681884766, 20.852096557617, 144.16523742676)
            QUESTTEXT = "Reward:\n$800\n1,800 Exp."
        elseif LEVEL == 15 or LEVEL <= 29 then
            MONNAME = "Gorilla [Lv. 20]"
            QUESTNAME = "JungleQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -1599.82532,
                36.8521347,
                153.959076,
                0.00275422214,
                5.1952032e-08,
                -0.999996185,
                1.33985356e-08,
                1,
                5.1989133e-08,
                0.999996185,
                -1.35416744e-08,
                0.00275422214
            )
            PUK = CFrame.new(-1123.5816650391, 40.462215423584, -499.97018432617)
            QUESTTEXT = "Reward:\n$1,200\n3,500 Exp."
        elseif LEVEL == 30 or LEVEL <= 39 then
            MONNAME = "Pirate [Lv. 35]"
            QUESTNAME = "BuggyQuest1"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -1139.89978,
                4.75204945,
                3827.28198,
                -0.978285253,
                -5.40383516e-09,
                0.207262978,
                8.35276559e-09,
                1,
                6.54975736e-08,
                -0.207262978,
                6.58065318e-08,
                -0.978285253
            )
            PUK = CFrame.new(-1050.1062011719, 67.651809692383, 3957.0092773438)
            QUESTTEXT = "Reward:\n$3,000\n10,000 Exp."
        elseif LEVEL == 40 or LEVEL <= 59 then
            MONNAME = "Brute [Lv. 45]"
            QUESTNAME = "BuggyQuest1"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -1139.89978,
                4.75204945,
                3827.28198,
                -0.978285253,
                -5.40383516e-09,
                0.207262978,
                8.35276559e-09,
                1,
                6.54975736e-08,
                -0.207262978,
                6.58065318e-08,
                -0.978285253
            )
            PUK = CFrame.new(-1153.0838623047, 57.161735534668, 4176.1440429688)
            QUESTTEXT = "Reward:\n$3,500\n18,000 Exp."
        elseif LEVEL == 60 or LEVEL <= 74 then
            MONNAME = "Desert Bandit [Lv. 60]"
            QUESTNAME = "DesertQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                896.408142,
                6.43846178,
                4389.37891,
                -0.846945703,
                -2.31297754e-08,
                0.531679392,
                -1.55507234e-08,
                1,
                1.87315088e-08,
                -0.531679392,
                7.59657048e-09,
                -0.846945703
            )
            PUK =
                CFrame.new(
                949.532593,
                15.2893953,
                4403.09912,
                -0.832314849,
                -4.19622452e-08,
                0.55430311,
                7.94023236e-10,
                1,
                7.68949704e-08,
                -0.55430311,
                6.44409539e-08,
                -0.832314849
            )
            QUESTTEXT = "Reward:\n$4,000\n35,000 Exp."
        elseif LEVEL == 75 or LEVEL <= 89 then
            MONNAME = "Desert Officer [Lv. 70]"
            QUESTNAME = "DesertQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                896.408142,
                6.43846178,
                4389.37891,
                -0.846945703,
                -2.31297754e-08,
                0.531679392,
                -1.55507234e-08,
                1,
                1.87315088e-08,
                -0.531679392,
                7.59657048e-09,
                -0.846945703
            )
            PUK =
                CFrame.new(
                1547.40369,
                14.4520388,
                4359.40625,
                0.228631318,
                -1.20783e-07,
                -0.973513126,
                -3.43095508e-08,
                1,
                -1.32126871e-07,
                0.973513126,
                6.36091286e-08,
                0.228631318
            )
            QUESTTEXT = "Reward:\n$4,500\n50,000 Exp."
        elseif LEVEL == 90 or LEVEL <= 99 then
            MONNAME = "Snow Bandit [Lv. 90]"
            QUESTNAME = "SnowQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                1384.14001,
                87.272789,
                -1297.06482,
                0.348555952,
                -2.53947841e-09,
                -0.937287986,
                1.49860568e-08,
                1,
                2.86358204e-09,
                0.937287986,
                -1.50443711e-08,
                0.348555952
            )
            PUK = CFrame.new(1419.1417236328, 119.61999511719, -1413.1545410156)
            QUESTTEXT = "Reward:\n$5,000\n70,000 Exp."
        elseif LEVEL == 100 or LEVEL <= 119 then
            MONNAME = "Snowman [Lv. 100]"
            QUESTNAME = "SnowQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                1384.14001,
                87.272789,
                -1297.06482,
                0.348555952,
                -2.53947841e-09,
                -0.937287986,
                1.49860568e-08,
                1,
                2.86358204e-09,
                0.937287986,
                -1.50443711e-08,
                0.348555952
            )
            PUK = CFrame.new(1219.9207763672, 138.01184082031, -1488.9234619141)
            QUESTTEXT = "Reward:\n$5,500\n120,000 Exp."
        elseif LEVEL == 120 or LEVEL <= 149 then
            MONNAME = "Chief Petty Officer [Lv. 120]"
            QUESTNAME = "MarineQuest2"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -5035.0835,
                28.6520386,
                4325.29443,
                0.0243340395,
                -7.08064647e-08,
                0.999703884,
                -6.36926814e-08,
                1,
                7.23777944e-08,
                -0.999703884,
                -6.54350671e-08,
                0.0243340395
            )
            PUK = CFrame.new(-4927.1708984375, 40.6520652771, 4152.5268554688)
            QUESTTEXT = "Reward:\n$6,000\n180,000 Exp."
        elseif LEVEL == 150 or LEVEL <= 174 then
            MONNAME = "Sky Bandit [Lv. 150]"
            QUESTNAME = "SkyQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -4841.83447,
                717.669617,
                -2623.96436,
                -0.875942111,
                5.59710216e-08,
                -0.482416272,
                3.04023082e-08,
                1,
                6.08195947e-08,
                0.482416272,
                3.86078725e-08,
                -0.875942111
            )
            PUK = CFrame.new(-4955.8520507813, 365.44644165039, -2911.5183105469)
            QUESTTEXT = "Reward:\n$7,000\n250,000 Exp."
        elseif LEVEL == 174 or LEVEL <= 224 then
            MONNAME = "Dark Master [Lv. 175]"
            QUESTNAME = "SkyQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -4841.83447,
                717.669617,
                -2623.96436,
                -0.875942111,
                5.59710216e-08,
                -0.482416272,
                3.04023082e-08,
                1,
                6.08195947e-08,
                0.482416272,
                3.86078725e-08,
                -0.875942111
            )
            PUK = CFrame.new(-5146.478515625, 439.27435302734, -2338.4731445313)
            QUESTTEXT = "Reward:\n$7,500\n350,000 Exp."
        elseif LEVEL == 225 or LEVEL <= 274 then
            MONNAME = "Toga Warrior [Lv. 225]"
            QUESTNAME = "ColosseumQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -1576.11743,
                7.38933945,
                -2983.30762,
                0.576966345,
                1.22114863e-09,
                0.816767931,
                -3.58496594e-10,
                1,
                -1.24185606e-09,
                -0.816767931,
                4.2370063e-10,
                0.576966345
            )
            PUK = CFrame.new(-1908.7215576172, 49.0544090271, -2902.6662597656)
            QUESTTEXT = "Reward:\n$7,000\n700,000 Exp."
        elseif LEVEL == 275 or LEVEL <= 299 then
            MONNAME = "Gladiator [Lv. 275]"
            QUESTNAME = "ColosseumQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -1576.11743,
                7.38933945,
                -2983.30762,
                0.576966345,
                1.22114863e-09,
                0.816767931,
                -3.58496594e-10,
                1,
                -1.24185606e-09,
                -0.816767931,
                4.2370063e-10,
                0.576966345
            )
            PUK = CFrame.new(-1491.9755859375, 49.053775787354, -3319.7390136719)
            QUESTTEXT = "Reward:\n$7,500\n1,000,000 Exp."
        elseif LEVEL == 300 or LEVEL <= 329 then
            MONNAME = "Military Soldier [Lv. 300]"
            QUESTNAME = "MagmaQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -5316.55859,
                12.2370615,
                8517.2998,
                0.588437557,
                -1.37880001e-08,
                -0.808542669,
                -2.10116209e-08,
                1,
                -3.23446478e-08,
                0.808542669,
                3.60215964e-08,
                0.588437557
            )
            PUK = CFrame.new(-5311.2822265625, 52.057613372803, 8501.89453125)
            QUESTTEXT = "Reward:\n$8,250\n1,300,000 Exp."
        elseif LEVEL == 330 or LEVEL <= 374 then
            MONNAME = "Military Spy [Lv. 330]"
            QUESTNAME = "MagmaQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -5316.55859,
                12.2370615,
                8517.2998,
                0.588437557,
                -1.37880001e-08,
                -0.808542669,
                -2.10116209e-08,
                1,
                -3.23446478e-08,
                0.808542669,
                3.60215964e-08,
                0.588437557
            )
            PUK = CFrame.new(-5786.09375, 120.39544677734, 8763.904296875)
            QUESTTEXT = "Reward:\n$8,500\n1,850,000 Exp."
        elseif LEVEL == 375 or LEVEL <= 399 then
            MONNAME = "Fishman Warrior [Lv. 375]"
            QUESTNAME = "FishmanQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                61122.5625,
                18.4716396,
                1568.16504,
                0.893533468,
                3.95251609e-09,
                0.448996574,
                -2.34327455e-08,
                1,
                3.78297464e-08,
                -0.448996574,
                -4.43233645e-08,
                0.893533468
            )
            PUK = CFrame.new(60998.36328125, 96.373001098633, 1409.1947021484)
            QUESTTEXT = "Reward:\n$8,750\n2,500,000 Exp."
        elseif LEVEL == 400 or LEVEL <= 449 then
            MONNAME = "Fishman Commando [Lv. 400]"
            QUESTNAME = "FishmanQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                61122.5625,
                18.4716396,
                1568.16504,
                0.893533468,
                3.95251609e-09,
                0.448996574,
                -2.34327455e-08,
                1,
                3.78297464e-08,
                -0.448996574,
                -4.43233645e-08,
                0.893533468
            )
            PUK = CFrame.new(61906.16796875, 108.50987243652, 1561.6348876953)
            QUESTTEXT = "Reward:\n$9,000\n3,000,000 Exp."
        elseif LEVEL == 450 or LEVEL <= 474 then
            MONNAME = "God's Guard [Lv. 450]"
            QUESTNAME = "SkyExp1Quest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -4721.71436,
                845.277161,
                -1954.20105,
                -0.999277651,
                -5.56969759e-09,
                0.0380011722,
                -4.14751478e-09,
                1,
                3.75035256e-08,
                -0.0380011722,
                3.73188307e-08,
                -0.999277651
            )
            PUK = CFrame.new(-4646.32421875, 930.53002929688, -1755.6791992188)
            QUESTTEXT = "Reward:\n$8,750\n3,800,000 Exp."
        elseif LEVEL == 475 or LEVEL <= 524 then
            MONNAME = "Shanda [Lv. 475]"
            QUESTNAME = "SkyExp1Quest"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-7861.2163085938, 5545.5180664063, -382.10601806641)
            PUK = CFrame.new(-7685.7646484375, 5582.951171875, -441.21575927734)
            QUESTTEXT = "Reward:\n$9,000\n4,300,000 Exp."
        elseif LEVEL == 525 or LEVEL <= 549 then
            MONNAME = "Royal Squad [Lv. 525]"
            QUESTNAME = "SkyExp2Quest"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-7904.7631835938, 5635.9897460938, -1411.6595458984)
            PUK = CFrame.new(-7640.3754882813, 5637.1079101563, -1410.9654541016)
            QUESTTEXT = "Reward:\n$9,500\n4,600,000 Exp."
        elseif LEVEL == 550 or LEVEL <= 624 then
            MONNAME = "Royal Soldier [Lv. 550]"
            QUESTNAME = "SkyExp2Quest"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-7904.7631835938, 5635.9897460938, -1411.6595458984)
            PUK = CFrame.new(-7859.8315429688, 5644.3002929688, -1706.5093994141)
            QUESTTEXT = "Reward:\n$9,750\n5,000,000 Exp."
        elseif LEVEL == 625 or LEVEL <= 649 then
            MONNAME = "Galley Pirate [Lv. 625]"
            QUESTNAME = "FountainQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                5254.60156,
                38.5011406,
                4049.69678,
                -0.0504891425,
                -3.62066501e-08,
                -0.998724639,
                -9.87921389e-09,
                1,
                -3.57534553e-08,
                0.998724639,
                8.06145284e-09,
                -0.0504891425
            )
            PUK = CFrame.new(5553.8735351563, 109.42530059814, 4081.8322753906)
            QUESTTEXT = "Reward:\n$10,000\n5,800,000 Exp."
        elseif LEVEL >= 650 then
            MONNAME = "Galley Captain [Lv. 650]"
            QUESTNAME = "FountainQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                5254.60156,
                38.5011406,
                4049.69678,
                -0.0504891425,
                -3.62066501e-08,
                -0.998724639,
                -9.87921389e-09,
                1,
                -3.57534553e-08,
                0.998724639,
                8.06145284e-09,
                -0.0504891425
            )
            PUK = CFrame.new(5562.802734375, 114.23865509033, 4813.486328125)
            QUESTTEXT = "Reward:\n$10,000\n6,300,000 Exp."
        end
    elseif game.PlaceId == 4442272183 then
        if LEVEL == 700 or LEVEL <= 724 then
            MONNAME = "Raider [Lv. 700]"
            QUESTNAME = "Area1Quest"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-426.83334350586, 72.99634552002, 1836.3073730469)
            PUK = CFrame.new(-308.35919189453, 109.8524017334, 2185.9724121094)
            QUESTTEXT = "Reward:\n$10,250\n6,750,000 Exp."
        elseif LEVEL == 725 or LEVEL <= 774 then
            MONNAME = "Mercenary [Lv. 725]"
            QUESTNAME = "Area1Quest"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-426.83334350586, 72.99634552002, 1836.3073730469)
            PUK = CFrame.new(-862.95141601563, 122.47104644775, 1431.6789550781)
            QUESTTEXT = "Reward:\n$10,500\n7,000,000 Exp."
        elseif LEVEL == 775 or LEVEL <= 799 then
            MONNAME = "Swan Pirate [Lv. 775]"
            QUESTNAME = "Area2Quest"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(636.67327880859, 73.096351623535, 918.58721923828)
            PUK = CFrame.new(975.18939208984, 142.72006225586, 1213.2492675781)
            QUESTTEXT = "Reward:\n$10,750\n7,500,000 Exp."
        elseif LEVEL == 800 or LEVEL <= 874 then
            MONNAME = "Factory Staff [Lv. 800]"
            QUESTNAME = "Area2Quest"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(636.67327880859, 73.096351623535, 918.58721923828)
            PUK = CFrame.new(346.08749389648, 139.86044311523, -247.63478088379)
            QUESTTEXT = "Reward:\n$11,000\n8,250,000 Exp."
        elseif LEVEL == 875 or LEVEL <= 899 then
            MONNAME = "Marine Lieutenant [Lv. 875]"
            QUESTNAME = "MarineQuest3"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -2442.65015,
                73.0511475,
                -3219.11523,
                -0.873540044,
                4.2329841e-08,
                -0.486752301,
                5.64383384e-08,
                1,
                -1.43220786e-08,
                0.486752301,
                -3.99823996e-08,
                -0.873540044
            )
            PUK = CFrame.new(-2880.888671875, 151.74026489258, -3102.7438964844)
            QUESTTEXT = "Reward:\n$11,250\n9,000,000 Exp."
        elseif LEVEL == 900 or LEVEL <= 949 then
            MONNAME = "Marine Captain [Lv. 900]"
            QUESTNAME = "MarineQuest3"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -2442.65015,
                73.0511475,
                -3219.11523,
                -0.873540044,
                4.2329841e-08,
                -0.486752301,
                5.64383384e-08,
                1,
                -1.43220786e-08,
                0.486752301,
                -3.99823996e-08,
                -0.873540044
            )
            PUK = CFrame.new(-2026.9750976563, 119.37310791016, -3183.2036132813)
            QUESTTEXT = "Reward:\n$11,500\n10,500,000 Exp."
        elseif LEVEL == 950 or LEVEL <= 974 then
            MONNAME = "Zombie [Lv. 950]"
            QUESTNAME = "ZombieQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -5492.79395,
                48.5151672,
                -793.710571,
                0.321800292,
                -6.24695815e-08,
                0.946807742,
                4.05616092e-08,
                1,
                5.21931227e-08,
                -0.946807742,
                2.16082796e-08,
                0.321800292
            )
            PUK = CFrame.new(-5563.548828125, 102.58236694336, -816.18304443359)
            QUESTTEXT = "Reward:\n$11,750\n11,000,000 Exp."
        elseif LEVEL == 975 or LEVEL <= 999 then
            MONNAME = "Vampire [Lv. 975]"
            QUESTNAME = "ZombieQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -5492.79395,
                48.5151672,
                -793.710571,
                0.321800292,
                -6.24695815e-08,
                0.946807742,
                4.05616092e-08,
                1,
                5.21931227e-08,
                -0.946807742,
                2.16082796e-08,
                0.321800292
            )
            PUK = CFrame.new(-6058.7250976563, 222.13470458984, -1269.2659912109)
            QUESTTEXT = "Reward:\n$12,000\n12,000,000 Exp."
        elseif LEVEL == 1000 or LEVEL <= 1049 then
            MONNAME = "Snow Trooper [Lv. 1000]"
            QUESTNAME = "SnowMountainQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                604.964966,
                401.457062,
                -5371.69287,
                0.353063971,
                1.89520435e-08,
                -0.935599446,
                -5.81846002e-08,
                1,
                -1.70033754e-09,
                0.935599446,
                5.50377841e-08,
                0.353063971
            )
            PUK = CFrame.new(401.12814331055, 453.24252319336, -5282.6806640625)
            QUESTTEXT = "Reward:\n$12,250\n13,000,000 Exp."
        elseif LEVEL == 1050 or LEVEL <= 1099 then
            MONNAME = "Winter Warrior [Lv. 1050]"
            QUESTNAME = "SnowMountainQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                604.964966,
                401.457062,
                -5371.69287,
                0.353063971,
                1.89520435e-08,
                -0.935599446,
                -5.81846002e-08,
                1,
                -1.70033754e-09,
                0.935599446,
                5.50377841e-08,
                0.353063971
            )
            PUK = CFrame.new(1254.4925537109, 460.20809936523, -5187.2954101563)
            QUESTTEXT = "Reward:\n$12,500\n14,200,000 Exp."
        elseif LEVEL == 1100 or LEVEL <= 1124 then
            MONNAME = "Lab Subordinate [Lv. 1100]"
            QUESTNAME = "IceSideQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -6060.10693,
                15.9868021,
                -4904.7876,
                -0.411000341,
                -5.06538868e-07,
                0.91163528,
                1.26306062e-07,
                1,
                6.12581289e-07,
                -0.91163528,
                3.66916197e-07,
                -0.411000341
            )
            PUK = CFrame.new(-5977.0205078125, 44.332748413086, -4609.603515625)
            QUESTTEXT = "Reward:\n$12,250\n15,500,000 Exp."
        elseif LEVEL == 1125 or LEVEL <= 1174 then
            MONNAME = "Horned Warrior [Lv. 1125]"
            QUESTNAME = "IceSideQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -6060.10693,
                15.9868021,
                -4904.7876,
                -0.411000341,
                -5.06538868e-07,
                0.91163528,
                1.26306062e-07,
                1,
                6.12581289e-07,
                -0.91163528,
                3.66916197e-07,
                -0.411000341
            )
            PUK = CFrame.new(-6403.0678710938, 36.843845367432, -5974.0307617188)
            QUESTTEXT = "Reward:\n$12,500\n16,800,000 Exp."
        elseif LEVEL == 1175 or LEVEL <= 1199 then
            MONNAME = "Magma Ninja [Lv. 1175]"
            QUESTNAME = "FireSideQuest"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-5427.5869140625, 15.977565765381, -5296.8896484375)
            PUK = CFrame.new(-5597.2622070313, 73.26025390625, -6048.6669921875)
            QUESTTEXT = "Reward:\n$12,250\n18,000,000 Exp."
        elseif LEVEL == 1200 or LEVEL <= 1249 then
            MONNAME = "Lava Pirate [Lv. 1200]"
            QUESTNAME = "FireSideQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -5431.09473,
                15.9868021,
                -5296.53223,
                0.831796765,
                1.15322464e-07,
                -0.555080295,
                -1.10814341e-07,
                1,
                4.17010995e-08,
                0.555080295,
                2.68240168e-08,
                0.831796765
            )
            PUK = CFrame.new(-5241.8046875, 51.883113861084, -4733.5087890625)
            QUESTTEXT = "Reward:\n$12,500\n20,000,000 Exp."
        elseif LEVEL == 1250 or LEVEL <= 1274 then
            MONNAME = "Ship Deckhand [Lv. 1250]"
            QUESTNAME = "ShipQuest1"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                1037.80127,
                125.092171,
                32911.6016,
                -0.244533166,
                -0,
                -0.969640911,
                -0,
                1.00000012,
                -0,
                0.96964103,
                0,
                -0.244533136
            )
            PUK = CFrame.new(915.94091796875, 125.05712890625, 32980.61328125)
            QUESTTEXT = "Reward:\n$12,250\n23,000,000 Exp."
        elseif LEVEL == 1275 or LEVEL <= 1299 then
            MONNAME = "Ship Engineer [Lv. 1275]"
            QUESTNAME = "ShipQuest1"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                1037.80127,
                125.092171,
                32911.6016,
                -0.244533166,
                -0,
                -0.969640911,
                -0,
                1.00000012,
                -0,
                0.96964103,
                0,
                -0.244533136
            )
            PUK = CFrame.new(909.83099365234, 93.184936523438, 32969.96484375)
            QUESTTEXT = "Reward:\n$12,500\n24,500,000 Exp."
        elseif LEVEL == 1300 or LEVEL <= 1324 then
            MONNAME = "Ship Steward [Lv. 1300]"
            QUESTNAME = "ShipQuest2"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                968.80957,
                125.092171,
                33244.125,
                -0.869560242,
                1.51905191e-08,
                -0.493826836,
                1.44108379e-08,
                1,
                5.38534195e-09,
                0.493826836,
                -2.43357912e-09,
                -0.869560242
            )
            PUK = CFrame.new(920.54138183594, 129.55603027344, 33440.90234375)
            QUESTTEXT = "Reward:\n$12,250\n26,500,000 Exp."
        elseif LEVEL == 1325 or LEVEL <= 1349 then
            MONNAME = "Ship Officer [Lv. 1325]"
            QUESTNAME = "ShipQuest2"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                968.80957,
                125.092171,
                33244.125,
                -0.869560242,
                1.51905191e-08,
                -0.493826836,
                1.44108379e-08,
                1,
                5.38534195e-09,
                0.493826836,
                -2.43357912e-09,
                -0.869560242
            )
            PUK = CFrame.new(904.19317626953, 181.43902587891, 33314.7578125)
            QUESTTEXT = "Reward:\n$12,500\n28,500,000 Exp."
        elseif LEVEL == 1350 or LEVEL <= 1374 then
            MONNAME = "Arctic Warrior [Lv. 1350]"
            QUESTNAME = "FrostQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                5669.43506,
                28.2117786,
                -6482.60107,
                0.888092756,
                1.02705066e-07,
                0.459664226,
                -6.20391774e-08,
                1,
                -1.03572376e-07,
                -0.459664226,
                6.34646895e-08,
                0.888092756
            )
            PUK = CFrame.new(5943.6411132813, 68.133918762207, -6123.7827148438)
            QUESTTEXT = "Reward:\n$12,250\n30,000,000 Exp."
        elseif LEVEL == 1375 or LEVEL <= 1424 then
            MONNAME = "Snow Lurker [Lv. 1375]"
            QUESTNAME = "FrostQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                5669.43506,
                28.2117786,
                -6482.60107,
                0.888092756,
                1.02705066e-07,
                0.459664226,
                -6.20391774e-08,
                1,
                -1.03572376e-07,
                -0.459664226,
                6.34646895e-08,
                0.888092756
            )
            PUK = CFrame.new(5526.6586914063, 257.11096191406, -6794.408203125)
            QUESTTEXT = "Reward:\n$12,500\n32,000,000 Exp."
        elseif LEVEL == 1425 or LEVEL <= 1449 then
            MONNAME = "Sea Soldier [Lv. 1425]"
            QUESTNAME = "ForgottenQuest"
            QUESTNUMBER = 1
            QUESTPOS =
                CFrame.new(
                -3052.99097,
                236.881363,
                -10148.1943,
                -0.997911751,
                4.42321983e-08,
                0.064591676,
                4.90968759e-08,
                1,
                7.37270085e-08,
                -0.064591676,
                7.67442998e-08,
                -0.997911751
            )
            PUK = CFrame.new(-3048.837890625, 6.1264486312866, -9748.853515625)
            QUESTTEXT = "Reward:\n$12,250\n33,500,000 Exp."
        elseif LEVEL >= 1450 then
            MONNAME = "Water Fighter [Lv. 1450]"
            QUESTNAME = "ForgottenQuest"
            QUESTNUMBER = 2
            QUESTPOS =
                CFrame.new(
                -3052.99097,
                236.881363,
                -10148.1943,
                -0.997911751,
                4.42321983e-08,
                0.064591676,
                4.90968759e-08,
                1,
                7.37270085e-08,
                -0.064591676,
                7.67442998e-08,
                -0.997911751
            )
            PUK = CFrame.new(-3156.7785644531, 282.26458740234, -10536.399414063)
            QUESTTEXT = "Reward:\n$12,500\n35,500,000 Exp."
        end
    else
        if LEVEL == 1450 or LEVEL <= 1524 then
            MONNAME = "Pirate Millionaire [Lv. 1500]"
            QUESTNAME = "PiratePortQuest"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-289.58728027344, 44.13646697998, 5579.5952148438)
            PUK = CFrame.new(-384.09393310547, 76.599105834961, 5583.2397460938)
            QUESTTEXT = "Reward:\n$13,000\n35,000,000 Exp."
        elseif LEVEL == 1525 or LEVEL <= 1574 then
            MONNAME = "Pistol Billionaire [Lv. 1525]"
            QUESTNAME = "PiratePortQuest"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-289.58728027344, 44.13646697998, 5579.5952148438)
            PUK = CFrame.new(-78.60083770752, 134.9169921875, 6062.7758789063)
            QUESTTEXT = "Reward:\n$15,000\n37,500,000 Exp."
        elseif LEVEL == 1575 or LEVEL <= 1599 then
            MONNAME = "Dragon Crew Warrior [Lv. 1575]"
            QUESTNAME = "AmazonQuest"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(5833.8427734375, 51.852348327637, -1102.8018798828)
            PUK = CFrame.new(6290.8100585938, 51.839542388916, -1219.7463378906)
            QUESTTEXT = "Reward:\n$13,000\n40,000,000 Exp."
        elseif LEVEL == 1600 or LEVEL <= 1624 then
            MONNAME = "Dragon Crew Archer [Lv. 1600]"
            QUESTNAME = "AmazonQuest"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(5833.8427734375, 51.852348327637, -1102.8018798828)
            PUK = CFrame.new(6631.734375, 383.95059204102, 164.15469360352)
            QUESTTEXT = "Reward:\n$15,000\n42,500,000 Exp."
        elseif LEVEL == 1624 or LEVEL <= 1649 then
            MONNAME = "Female Islander [Lv. 1625]"
            QUESTNAME = "AmazonQuest2"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(5447.4189453125, 602.09130859375, 750.89605712891)
            PUK = CFrame.new(4557.26953125, 838.26525878906, 672.03967285156)
            QUESTTEXT = "Reward:\n$13,000\n45,500,000 Exp."
        elseif LEVEL == 1650 or LEVEL <= 1699 then
            MONNAME = "Giant Islander [Lv. 1650]"
            QUESTNAME = "AmazonQuest2"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(5447.4189453125, 602.09130859375, 750.89605712891)
            PUK = CFrame.new(4917.2124023438, 670.27093505859, -1.021057844162)
            QUESTTEXT = "Reward:\n$15,000\n48,000,000 Exp."
        elseif LEVEL == 1700 or LEVEL <= 1724 then
            MONNAME = "Marine Commodore [Lv. 1700]"
            QUESTNAME = "MarineTreeIsland"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(2180.0053710938, 29.048696517944, -6739.6196289063)
            PUK = CFrame.new(2574.1418457031, 366.51287841797, -7232.9975585938)
            QUESTTEXT = "Reward:\n$13,000\n51,000,000 Exp."
        elseif LEVEL == 1725 or LEVEL <= 1774 then
            MONNAME = "Marine Rear Admiral [Lv. 1725]"
            QUESTNAME = "MarineTreeIsland"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(2180.0053710938, 29.048696517944, -6739.6196289063)
            PUK = CFrame.new(3635.4694824219, 160.86730957031, -6976.375)
            QUESTTEXT = "Reward:\n$15,000\n53,500,000 Exp."
        elseif LEVEL == 1775 or LEVEL <= 1799 then
            MONNAME = "Fishman Raider [Lv. 1775]"
            QUESTNAME = "DeepForestIsland3"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-10582.411132813, 332.10589599609, -8759.796875)
            PUK = CFrame.new(-10634.860351563, 332.10589599609, -8485.3505859375)
            QUESTTEXT = "Reward:\n$13,000\n56,000,000 Exp."
        elseif LEVEL == 1800 or LEVEL <= 1824 then
            MONNAME = "Fishman Captain [Lv. 1800]"
            QUESTNAME = "DeepForestIsland3"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-10582.411132813, 332.10589599609, -8759.796875)
            PUK = CFrame.new(-10933.125, 332.10589599609, -8878.7431640625)
            QUESTTEXT = "Reward:\n$15,000\n58,500,000 Exp."
        elseif LEVEL == 1825 or LEVEL <= 1849 then
            MONNAME = "Forest Pirate [Lv. 1825]"
            QUESTNAME = "DeepForestIsland"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-13232.396484375, 332.7214050293, -7625.89453125)
            PUK = CFrame.new(-13461.088867188, 415.26861572266, -7799.16796875)
            QUESTTEXT = "Reward:\n$13,000\n61,000,000 Exp."
        elseif LEVEL == 1850 or LEVEL <= 1899 then
            MONNAME = "Mythological Pirate [Lv. 1850]"
            QUESTNAME = "DeepForestIsland"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-13232.396484375, 332.7214050293, -7625.89453125)
            PUK = CFrame.new(-13509.658203125, 583.88702392578, -6987.2329101563)
            QUESTTEXT = "Reward:\n$13,000\n64,000,000 Exp."
        elseif LEVEL == 1900 or LEVEL <= 1924 then
            MONNAME = "Jungle Pirate [Lv. 1900]"
            QUESTNAME = "DeepForestIsland2"
            QUESTNUMBER = 1
            QUESTPOS = CFrame.new(-12681.626953125, 391.76815795898, -9901.892578125)
            PUK = CFrame.new(-12097.916992188, 455.87445068359, -10583.470703125)
            QUESTTEXT = "Reward:\n$13,000\n67,000,000 Exp."
        elseif LEVEL >= 1925 then
            MONNAME = "Musketeer Pirate [Lv. 1925]"
            QUESTNAME = "DeepForestIsland2"
            QUESTNUMBER = 2
            QUESTPOS = CFrame.new(-12681.626953125, 391.76815795898, -9901.892578125)
            PUK = CFrame.new(-13059.739257813, 443.66815185547, -9797.1689453125)
            QUESTTEXT = "Reward:\n$15,000\n70,000,000 Exp."
        end
    end
end

function ChangeMethodFarm()
    if HHHHHHHHH == 1 then
        MethodFarmAAAAAAAAAAAAAA = CFrame.new(0, 15, 15)
        HHHHHHHHH = 2
    else
        MethodFarmAAAAAAAAAAAAAA = CFrame.new(0, 15, -15)
        HHHHHHHHH = 1
    end
end

if game.CoreGui:FindFirstChild(NAME_OF_HUB) then
    game.CoreGui:FindFirstChild(NAME_OF_HUB):Destroy()
end

local venyx = library.new(NAME_OF_HUB, 5013109572)

local page = venyx:addPage("Main", 5012544693)
local section1 = page:addSection("Auto Farm")
local section2 = page:addSection("Auto Equip")
local section4 = page:addSection("Auto Farm Boss")
local page4 = venyx:addPage("Stats", 5012544693)
local s1 = page4:addSection("Auto Stats")
local page2 = venyx:addPage("Players", 5012544693)
local lplr = page2:addSection("Destroy")
local llplr = page2:addSection("Local")
local page3 = venyx:addPage("Teleport", 5012544693)
local ww = page3:addSection("Teleport world")
local tp = page3:addSection("Teleport Island")
local page5 = venyx:addPage("Dungion", 5012544693)
local raid = page5:addSection("Raid")
local page6 = venyx:addPage("Devil Fruit", 5012544693)
local df2 = page6:addSection("Grab Devil Fruit")


section1:addToggle(
    "Auto Farm Level",
    _G.AutoFarm_Level,
    function(value)
        _G.AutoFarm_Level = value
    end
)

spawn(
    function()
        while true do
            wait(.01)
            CheckLevel()
            if _G.AutoFarm_Level then
                if game:GetService("Workspace").Enemies:FindFirstChild(MONNAME) then
                else
                    CheckLevel()
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = PUK
                end
                autohaki()
                pcall(
                    function()
                        CheckLevel()
                        require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework.CameraShaker).CameraShakeInstance.CameraShakeState = {
                            FadingIn = 3,
                            FadingOut = 2,
                            Sustained = 0,
                            Inactive = 1
                        }

                        if
                            game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestReward.Title.Text ==
                                QUESTTEXT
                         then
                        else
                            AbandonQ()
                        end

                        if game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible ~= true then
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = QUESTPOS
                            wait(1)
                            GetQuest(QUESTNAME, QUESTNUMBER)
                            wait(1)
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = PUK
                        end

                        for i, v in pairs(game.Workspace.Enemies:GetChildren()) do
                            if v.Name == MONNAME then
                                game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
                                if
                                    tostring(game:GetService("Players").LocalPlayer.Character.Humanoid.FloorMaterial) ~=
                                        "Enum.Material.Air"
                                 then
                                    ChangeMethodFarm()
                                end
                                Teleport(v.HumanoidRootPart.CFrame * MethodFarmAAAAAAAAAAAAAA)
                                game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
                                for ii, vv in pairs(game.Workspace.Enemies:GetChildren()) do
                                    if
                                        v.Name == MONNAME and vv.Name == MONNAME and v.Humanoid.Health == 0 or
                                            vv.Humanoid.Health == 0
                                     then
                                    elseif v.Name == MONNAME and vv.Name == MONNAME then
                                        vv.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame
                                        v.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                                        vv.HumanoidRootPart.Size = Vector3.new(50, 50, 50)
                                        vv.HumanoidRootPart.CanCollide = false
                                        v.Humanoid.WalkSpeed = 0
                                        vv.Humanoid.WalkSpeed = 0
                                        v.HumanoidRootPart.CanCollide = false
                                        v.Humanoid:ChangeState(11)
                                        vv.Humanoid:ChangeState(11)
                                        sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
                                        require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework).activeController.hitboxMagnitude =
                                            60
                                    end
                                end
                            end
                        end

                        Attack()
                    end
                )
            end
        end
    end
)

section1:addToggle("Fast Attack",_G.Fast_Attack,function(value)
        _G.Fast_Attack = value
end)



section1:addToggle("Auto Saber",_G.Auto_Saber,function(value)
        _G.Auto_Saber = value
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-1419.4379882813, 47.852024078369, 12.422969818115)
    while _G.Auto_Saber do wait()
    
        if game:GetService("Workspace").Map.Jungle.Final.Part.Transparency == 1 then
        
            for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
                if v.Name == "Saber Expert [Lv. 200] [Boss]" and v.Humanoid.Health <= 0 then
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-1419.4379882813, 47.852024078369, 12.422969818115)
                    elseif v.Name == "Saber Expert [Lv. 200] [Boss]" then
                    v.HumanoidRootPart.Size = Vector3.new(60,60,60)
                    game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0,20,0)
                    game:GetService'VirtualUser':CaptureController()
                game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
                    end
                end
            
            else
                function EquipWeapon(ToolSe)
           if game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe) then
              local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe)
              wait(.4)
              game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
           end
        end
        
    
    
    
    
        
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game:GetService("Workspace").Map.Jungle.QuestPlates.Plate1.Button.CFrame
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game:GetService("Workspace").Map.Jungle.QuestPlates.Plate2.Button.CFrame
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game:GetService("Workspace").Map.Jungle.QuestPlates.Plate3.Button.CFrame
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game:GetService("Workspace").Map.Jungle.QuestPlates.Plate4.Button.CFrame
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game:GetService("Workspace").Map.Jungle.QuestPlates.Plate5.Button.CFrame
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-1609.505859375, 12.05205821991, 162.31004333496)
        local args = {
            [1] = "ProQuestProgress",
            [2] = "GetTorch"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-1609.505859375, 12.05205821991, 162.31004333496)
        wait(.3)
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-1609.505859375, 12.05205821991, 162.31004333496)
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1115.40466, 4.92146635, 4349.39941, -0.683263719, -6.49155467e-08, 0.73017168, -1.55561808e-09, 1, 8.74488109e-08, -0.73017168, 5.86147344e-08, -0.683263719)
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1115.40466, 4.92146635, 4349.39941, -0.683263719, -6.49155467e-08, 0.73017168, -1.55561808e-09, 1, 8.74488109e-08, -0.73017168, 5.86147344e-08, -0.683263719)
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1115.40466, 4.92146635, 4349.39941, -0.683263719, -6.49155467e-08, 0.73017168, -1.55561808e-09, 1, 8.74488109e-08, -0.73017168, 5.86147344e-08, -0.683263719)
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1115.40466, 4.92146635, 4349.39941, -0.683263719, -6.49155467e-08, 0.73017168, -1.55561808e-09, 1, 8.74488109e-08, -0.73017168, 5.86147344e-08, -0.683263719)
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1115.40466, 4.92146635, 4349.39941, -0.683263719, -6.49155467e-08, 0.73017168, -1.55561808e-09, 1, 8.74488109e-08, -0.73017168, 5.86147344e-08, -0.683263719)
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1115.40466, 4.92146635, 4349.39941, -0.683263719, -6.49155467e-08, 0.73017168, -1.55561808e-09, 1, 8.74488109e-08, -0.73017168, 5.86147344e-08, -0.683263719)
        wait(1)
        EquipWeapon("Torch")
        wait(14)
        
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1113.6097412109, 5.2971863746643, 4364.9819335938)
        
        local args = {
            [1] = "ProQuestProgress",
            [2] = "GetCup"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        
        
        wait()
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1113.6097412109, 5.2971863746643, 4364.9819335938)
        wait()
        EquipWeapon("Cup")
        wait(2)
        local args = {
            [1] = "ProQuestProgress",
            [2] = "FillCup",
            [3] = game:GetService("Players").LocalPlayer.Character.Cup
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        wait(3)
        
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1459.9223632813, 88.660217285156, -1389.4007568359)
        local args = {
            [1] = "ProQuestProgress",
            [2] = "SickMan"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        wait(2)
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-909.64031982422, 13.752033233643, 4077.7365722656)
        wait()
        local args = {
            [1] = "ProQuestProgress",
            [2] = "RichSon"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        PUKMOB = CFrame.new(-2890.3310546875, 49.201908111572, 5424.435546875)
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = PUKMOB
        ABC = false
        
        repeat
            
            wait()
            for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
            if v.Name == "Mob Leader [Lv. 120] [Boss]" and v.Humanoid.Health <= 0 then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = PUKMOB
            _G.NEXT = true
            ABC = true
            elseif v.Name == "Mob Leader [Lv. 120] [Boss]" then
                v.HumanoidRootPart.Size = Vector3.new(60,60,60)
                game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0,20,0)
                game:GetService'VirtualUser':CaptureController()
                game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
            end
            end
        until ABC
        
        
            
            spawn(function()
                while wait(.1) do 
                    if _G.NEXT then 
                   _G.NEXT = false
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-909.64031982422, 13.752033233643, 4077.7365722656)
        
        local args = {
            [1] = "ProQuestProgress",
            [2] = "RichSon"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        
        local args = {
            [1] = "ProQuestProgress"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        wait(1)
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-1405.86304, 29.8519974, 4.36262178, 0.863304675, 3.85057408e-09, 0.504683077, 5.51599193e-08, 1, -1.0198557e-07, -0.504683077, 1.15882898e-07, 0.863304675)
        EquipWeapon("Relic")
        wait()
        local args = {
            [1] = "ProQuestProgress",
            [2] = "PlaceRelic"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        local args = {
            [1] = "ProQuestProgress"
        }
        
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
        
                end
                end
                end)
        end
    end
end)

spawn(
    function()
        pcall(
            function()
                game:GetService("RunService").Heartbeat:Connect(
                    function()
                        if _G.Fast_Attack then
                            Rig = require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework)
                            Cam =
                                require(
                                game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework.CameraShaker
                            )
                            Rig.activeController.timeToNextAttack = 1632600095.36
                            for i = 1, 3 do
                                Cam.CameraShakeInstance.CameraShakeState = {
                                    FadingIn = 3,
                                    FadingOut = 2,
                                    Sustained = 0,
                                    Inactive = 1
                                }
                                if _G.AutoFarm_Level or KILL or _G.Auto_Saber or _G.AFB then
                                    Cam.CameraShakeInstance.CameraShakeState = {
                                        FadingIn = 3,
                                        FadingOut = 2,
                                        Sustained = 0,
                                        Inactive = 1
                                    }
                                    Rig.activeController.timeToNextAttack = 0
                                    game:GetService "VirtualUser":CaptureController()
                                    game:GetService "VirtualUser":Button1Down(Vector2.new(1280, 672))

                                    Cam.CameraShakeInstance.CameraShakeState = {
                                        FadingIn = 3,
                                        FadingOut = 2,
                                        Sustained = 0,
                                        Inactive = 1
                                    }
                                end
                            end
                        end
                    end
                )
            end
        )
    end
)

section2:addToggle("Auto Equipped", _G.EQP01, function(value)
        _G.EQP01 = value
    end)

    spawn(function()
while wait(.01) do
    if _G.EQP01 then
        pcall(function()
            if _G.WEAPONSELECT == nil then
                venyx:Notify("Error!!", "Please, Select Weapon !!")
                _G.EQP01 = false
                else
            EquipWeapon(_G.WEAPONSELECT)
            end
        end)
    end
end

    end)
    
    local WEAPON = {}
    
    for i,v in pairs(game:GetService("Players").LocalPlayer.Backpack:GetChildren()) do
    if v:IsA("Tool") then
        table.insert(WEAPON,v.Name)
        end
    end
    
    section2:addDropdown("Select Weapon", WEAPON, function(a)
       _G.WEAPONSELECT = a
    end)
    
    section2:addButton("Refresh Weapon", function()
        table.clear(WEAPON)
    for i,v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
    if v:IsA("Tool") then
        table.insert(WEAPON,v.Name)
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
    
            section4:addDropdown("Select Boss", Boss, function(Value)
                _G.SELECTBOSS = Value
    
            end)
    
    
    
    
            section4:addButton("Refresh Boss", function()
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
            end)
    
            section4:addToggle("Get Quest", nil, function(value)
                _G.GETQUESTBOSS = value
            end)

            section4:addToggle("Auto Farm Boss", nil, function(value)
                _G.AFB = value
    
    while _G.AFB do wait()
        pcall(function()
            function checkbossselect()
        if _G.SELECTBOSS == "Warden [Lv. 175] [Boss]" then --warden
            _G.QUESTPOSBOSS = CFrame.new(4854.9887695313, 5.6784439086914, 745.75201416016)
            _G.PUKPOSBOSS = CFrame.new(5271.8110351563, 161.83946228027, 843.14056396484)
            _G.NAMEMONBOSS = "Warden [Lv. 175] [Boss]"
            _G.QUESTNAMEBOSS = "ImpelQuest"
            _G.QUESTNUMBOSS = 1
            _G.REWARDBOSS = "Reward:\n$6,000\n600,000 Exp."
        elseif _G.SELECTBOSS == "Chief Warden [Lv. 200] [Boss]" then --chief warden
            _G.QUESTPOSBOSS = CFrame.new(4854.9887695313, 5.6784439086914, 745.75201416016)
            _G.PUKPOSBOSS = CFrame.new(5271.8110351563, 161.83946228027, 843.14056396484)
            _G.NAMEMONBOSS = "Chief Warden [Lv. 200] [Boss]"
            _G.QUESTNAMEBOSS = "ImpelQuest"
            _G.QUESTNUMBOSS = 2
            _G.REWARDBOSS = "Reward:\n$10,000\n700,000 Exp."
        elseif _G.SELECTBOSS == "Swan [Lv. 225] [Boss]" then --swan
            _G.QUESTPOSBOSS = CFrame.new(4854.9887695313, 5.6784439086914, 745.75201416016)
            _G.PUKPOSBOSS = CFrame.new(5271.8110351563, 161.83946228027, 843.14056396484)
            _G.NAMEMONBOSS = "Swan [Lv. 225] [Boss]"
            _G.QUESTNAMEBOSS = "ImpelQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$15,000\n1,300,000 Exp."
        elseif _G.SELECTBOSS == "The Gorilla King [Lv. 25] [Boss]" then --gorilla
            _G.QUESTPOSBOSS = CFrame.new(-1600.2290039063, 36.8779296875, 153.01272583008)
            _G.PUKPOSBOSS = CFrame.new(-1127.2087402344, 74.843826293945, -511.47674560547)
            _G.NAMEMONBOSS = "The Gorilla King [Lv. 25] [Boss]"
            _G.QUESTNAMEBOSS = "JungleQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$2,000\n7,000 Exp."
        elseif _G.SELECTBOSS == "Vice Admiral [Lv. 130] [Boss]" then --vice aminal
            _G.QUESTPOSBOSS = CFrame.new(-5037.822265625, 28.677835464478, 4324.57421875)
            _G.PUKPOSBOSS = CFrame.new(-5113.41796875, 169.67807006836, 4365.59375)
            _G.NAMEMONBOSS = "Vice Admiral [Lv. 130] [Boss]"
            _G.QUESTNAMEBOSS = "MarineQuest2"
            _G.QUESTNUMBOSS = 2
            _G.REWARDBOSS = "Reward:\n$15,000\n350,000 Exp."
        elseif _G.SELECTBOSS == "Wysper [Lv. 500] [Boss]" then -- Wysper
            _G.QUESTPOSBOSS = CFrame.new(-7862.94629, 5545.52832, -379.833954, 0.462944925, 1.45838088e-08, -0.886386991, 1.0534996e-08, 1, 2.19553424e-08, 0.886386991, -1.95022007e-08, 0.462944925)
            _G.PUKPOSBOSS = CFrame.new(-7989.6337890625, 5545.6357421875, -520.77874755859)
            _G.NAMEMONBOSS = "Wysper [Lv. 500] [Boss]"
            _G.QUESTNAMEBOSS = "SkyExp1Quest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$15,000\n4,800,000 Exp."
        elseif _G.SELECTBOSS == "Magma Admiral [Lv. 350] [Boss]" then -- magma aminal
            _G.QUESTPOSBOSS = CFrame.new(-5317.07666, 12.2721891, 8517.41699, 0.51175487, -2.65508806e-08, -0.859131515, -3.91131572e-08, 1, -5.42026761e-08, 0.859131515, 6.13418294e-08, 0.51175487)
            _G.PUKPOSBOSS = CFrame.new(-5545.8940429688, 176.00048828125, 8657.134765625)
            _G.NAMEMONBOSS = "Magma Admiral [Lv. 350] [Boss]"
            _G.QUESTNAMEBOSS = "MagmaQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$15,000\n2,800,000 Exp."
        elseif _G.SELECTBOSS == "Yeti [Lv. 110] [Boss]" then -- Yeti
            _G.QUESTPOSBOSS = CFrame.new(1387.6511230469, 87.298568725586, -1298.0422363281)
            _G.PUKPOSBOSS = CFrame.new(1220.3798828125, 138.03759765625, -1489.2705078125)
            _G.NAMEMONBOSS = "Yeti [Lv. 110] [Boss]"
            _G.QUESTNAMEBOSS = "SnowQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$10,000\n180,000 Exp."
        elseif _G.SELECTBOSS == "Bobby [Lv. 55] [Boss]" then -- buggy
            _G.QUESTPOSBOSS = CFrame.new(-1140.1296386719, 5.177855014801, 3830.4733886719)
            _G.PUKPOSBOSS = CFrame.new(-1152.2501220703, 57.187534332275, 4174.9418945313)
            _G.NAMEMONBOSS = "Bobby [Lv. 55] [Boss]"
            _G.QUESTNAMEBOSS = "BuggyQuest1"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$8,000\n35,000 Exp."
        elseif _G.SELECTBOSS == "Fishman Lord [Lv. 425] [Boss]" then -- fis lord
            _G.QUESTPOSBOSS = CFrame.new(61121.74609375, 18.497438430786, 1566.8347167969)
            _G.PUKPOSBOSS = CFrame.new(61351.109375, 103.18898010254, 1244.9989013672)
            _G.NAMEMONBOSS = "Fishman Lord [Lv. 425] [Boss]"
            _G.QUESTNAMEBOSS = "FishmanQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$15,000\n4,000,000 Exp."
        elseif _G.SELECTBOSS == "Cyborg [Lv. 675] [Boss]" then -- cybrog
            _G.QUESTPOSBOSS = CFrame.new(5258.2646484375, 38.526931762695, 4049.0847167969)
            _G.PUKPOSBOSS = CFrame.new(6098.583984375, 59.527156829834, 4058.7717285156)
            _G.NAMEMONBOSS = "Cyborg [Lv. 675] [Boss]"
            _G.QUESTNAMEBOSS = "FountainQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$20,000\n7,500,000 Exp."
        elseif _G.SELECTBOSS == "The Saw [Lv. 100] [Boss]" then -- cThe Saw
            _G.PUKPOSBOSS = CFrame.new(-807.36236572266, 73.867568969727, 1613.8154296875)
            _G.NAMEMONBOSS = "The Saw [Lv. 100] [Boss]"
        elseif _G.SELECTBOSS == "Mob Leader [Lv. 120] [Boss]" then -- Mob Leader [Lv. 120] [Boss]
            _G.PUKPOSBOSS = CFrame.new(-2881.6022949219, 48.577754974365, 5411.3466796875)
            _G.NAMEMONBOSS = "Mob Leader [Lv. 120] [Boss]"
        elseif _G.SELECTBOSS == "Thunder God [Lv. 575] [Boss]" then -- Enel
            _G.QUESTPOSBOSS = CFrame.new(-7905.1743164063, 5635.9887695313, -1411.2357177734)
            _G.PUKPOSBOSS = CFrame.new(-7995.958984375, 5756.0576171875, -2084.2373046875)
            _G.NAMEMONBOSS = "Thunder God [Lv. 575] [Boss]"
            _G.QUESTNAMEBOSS = "SkyExp2Quest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$20,000\n5,800,000 Exp."
        elseif _G.SELECTBOSS == "Jeremy [Lv. 850] [Boss]" then -- jeramy
            _G.QUESTPOSBOSS = CFrame.new(635.98907470703, 73.096328735352, 918.51379394531)
            _G.PUKPOSBOSS = CFrame.new(2110.5737304688, 448.95666503906, 573.408325195311)
            _G.NAMEMONBOSS = "Jeremy [Lv. 850] [Boss]"
            _G.QUESTNAMEBOSS = "Area2Quest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$25,000\n11,500,000 Exp."
        elseif _G.SELECTBOSS == "Diamond [Lv. 750] [Boss]" then -- diamond
            _G.QUESTPOSBOSS = CFrame.new(-428.22674560547, 72.996322631836, 1836.9592285156)
            _G.PUKPOSBOSS = CFrame.new(-1743.6767578125, 198.70874023438, -58.437126159668)
            _G.NAMEMONBOSS = "Diamond [Lv. 750] [Boss]"
            _G.QUESTNAMEBOSS = "Area1Quest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$25,000\n9,000,000 Exp."
        elseif _G.SELECTBOSS == "Fajita [Lv. 925] [Boss]" then -- fujita
            _G.QUESTPOSBOSS = CFrame.new(-2442.5795898438, 73.041854858398, -3216.841796875)
            _G.PUKPOSBOSS = CFrame.new(-2336.7504882813, 73.621597290039, -4272.4375)
            _G.NAMEMONBOSS = "Fajita [Lv. 925] [Boss]"
            _G.QUESTNAMEBOSS = "MarineQuest3"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$25,000\n15,000,000 Exp."
        elseif _G.SELECTBOSS == "Cursed Captain [Lv. 1325] [Raid Boss]" then -- Cursed
            _G.PUKPOSBOSS = CFrame.new(877.60272216797, 181.46482849121, 33301.6640625)
            _G.NAMEMONBOSS = "Cursed Captain [Lv. 1325] [Raid Boss]"
        elseif _G.SELECTBOSS == "Tide Keeper [Lv. 1475] [Boss]" then -- Tide
            _G.QUESTPOSBOSS = CFrame.new(-3053.734375, 236.87208557129, -10145.025390625)
            _G.PUKPOSBOSS = CFrame.new(-3795.359375, 105.88877105713, -11407.432617188)
            _G.NAMEMONBOSS = "Tide Keeper [Lv. 1475] [Boss]"
            _G.QUESTNAMEBOSS = "ForgottenQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$12,500\n38,000,000 Exp."
        elseif _G.SELECTBOSS == "Awakened Ice Admiral [Lv. 1400] [Boss]" then -- awake ice
            _G.QUESTPOSBOSS = CFrame.new(5668.0419921875, 28.202545166016, -6483.861328125)
            _G.PUKPOSBOSS = CFrame.new(6400.8500976563, 340.43240356445, -6896.9921875)
            _G.NAMEMONBOSS = "Awakened Ice Admiral [Lv. 1400] [Boss]"
            _G.QUESTNAMEBOSS = "FrostQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$20,000\n36,000,000 Exp."
        elseif _G.SELECTBOSS == "Smoke Admiral [Lv. 1150] [Boss]" then -- Smoke Admiral
            _G.QUESTPOSBOSS = CFrame.new(-6062.8588867188, 15.977560997009, -4904.6127929688)
            _G.PUKPOSBOSS = CFrame.new(-5089.7158203125, 188.50866699219, -5346.1313476563)
            _G.NAMEMONBOSS = "Smoke Admiral [Lv. 1150] [Boss]"
            _G.QUESTNAMEBOSS = "IceSideQuest"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$20,000\n25,000,000 Exp."
        elseif _G.SELECTBOSS == "Don Swan [Lv. 1000] [Boss]" then -- Cursed
            _G.PUKPOSBOSS = CFrame.new(2284.3132324219, 15.177842140198, 879.32080078125)
            _G.NAMEMONBOSS = "Don Swan [Lv. 1000] [Boss]"
        elseif _G.SELECTBOSS == "Saber Expert [Lv. 200] [Boss]" then -- Cursed
            _G.PUKPOSBOSS = CFrame.new(-1419.4379882813, 47.852024078369, 12.422969818115)
            _G.NAMEMONBOSS = "Saber Expert [Lv. 200] [Boss]"
        elseif _G.SELECTBOSS == "Order [Lv. 1250] [Raid Boss]" then -- Cursed
           _G.PUKPOSBOSS = CFrame.new(-6408.0727539063, 305.88467407227, -4954.5786132813)
            _G.NAMEMONBOSS = "Order [Lv. 1250] [Raid Boss]"
            
        elseif _G.SELECTBOSS == "Kilo Admiral [Lv. 1750] [Boss]" then -- Kilo
            _G.QUESTPOSBOSS = CFrame.new(2180.0922851563, 29.048696517944, -6739.7255859375)
            _G.PUKPOSBOSS = CFrame.new(2889.4248046875, 457.19549560547, -7377.3466796875)
            _G.NAMEMONBOSS = "Kilo Admiral [Lv. 1750] [Boss]"
            _G.QUESTNAMEBOSS = "MarineTreeIsland"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$35,000\n56,000,000 Exp."
        elseif _G.SELECTBOSS == "Captain Elephant [Lv. 1875] [Boss]" then -- elephant
            _G.QUESTPOSBOSS = CFrame.new(-13232.56640625, 332.72137451172, -7626.6176757813)
            _G.PUKPOSBOSS = CFrame.new(-13229.93359375, 332.7214050293, -7626.3227539063)
            _G.NAMEMONBOSS = "Captain Elephant [Lv. 1875] [Boss]"
            _G.QUESTNAMEBOSS = "DeepForestIsland"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$40,000\n67,000,000 Exp."
        elseif _G.SELECTBOSS == "Longma [Lv. 2000] [Boss]" then -- longma
            _G.PUKPOSBOSS = CFrame.new(-10183.750976563, 375.10870361328, -9528.703125)
            _G.NAMEMONBOSS = "Longma [Lv. 2000] [Boss]"
        elseif _G.SELECTBOSS == "Island Empress [Lv. 1675] [Boss]" then -- elephant
            _G.QUESTPOSBOSS = CFrame.new(5447.00390625, 601.94689941406, 751.08575439453)
            _G.PUKPOSBOSS = CFrame.new(5822.9213867188, 662.63671875, 206.12593078613)
            _G.NAMEMONBOSS = "Island Empress [Lv. 1675] [Boss]"
            _G.QUESTNAMEBOSS = "AmazonQuest2"
            _G.QUESTNUMBOSS = 3
            _G.REWARDBOSS = "Reward:\n$30,000\n52,000,000 Exp."
    
        elseif _G.SELECTBOSS == "rip_indra True Form [Lv. 5000] [Raid Boss]" then -- rip indra
            _G.PUKPOSBOSS = CFrame.new(-5333.3090820313, 424.32867431641, -2667.6750488281)
            _G.NAMEMONBOSS = "rip_indra True Form [Lv. 5000] [Raid Boss]"
        elseif _G.SELECTBOSS == "Beautiful Pirate [Lv. 1950] [Boss]" then -- longma
            _G.PUKPOSBOSS = CFrame.new(5313.85889, 22.5622349, -124.438652, -0.997363091, 8.28968965e-08, -0.0725729316, 8.24298425e-08, 1, 9.43069001e-09, 0.0725729316, 3.42364692e-09, -0.997363091)
            _G.NAMEMONBOSS = "Beautiful Pirate [Lv. 1950] [Boss]"
    
    
    
    
            
    
            
            
    end
end
    
    
checkbossselect()
    
    
    
    if _G.SELECTBOSS == "The Saw [Lv. 100] [Boss]" or _G.SELECTBOSS == "Mob Leader [Lv. 120] [Boss]" or _G.SELECTBOSS == "Order [Lv. 1250] [Raid Boss]"  or  _G.SELECTBOSS == "Cursed Captain [Lv. 1325] [Raid Boss]" or _G.SELECTBOSS ==  "Don Swan [Lv. 1000] [Boss]" or _G.SELECTBOSS == "Saber Expert [Lv. 200] [Boss]" or _G.SELECTBOSS == "Longma [Lv. 2000] [Boss]" or _G.SELECTBOSS == "Beautiful Pirate [Lv. 1950] [Boss]" or _G.SELECTBOSS == "rip_indra True Form [Lv. 5000] [Raid Boss]" then
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.PUKPOSBOSS
    else
    if _G.GETQUESTBOSS then 
    if game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Container.QuestReward.Title.Text == _G.REWARDBOSS then
    
    else
    local args = {
        [1] = "AbandonQuest"
    }
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.QUESTPOSBOSS
    wait(1.5)
    checkbossselect()
    local args = {
        [1] = "StartQuest",
        [2] = _G.QUESTNAMEBOSS,
        [3] = _G.QUESTNUMBOSS
    }
    
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    wait(1.5)
    checkbossselect()
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.PUKPOSBOSS
    end
    
    
    if game:GetService("Players").LocalPlayer.PlayerGui.Main.Quest.Visible == true then
    
    else
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.QUESTPOSBOSS
    wait(1.5)
    checkbossselect()
    local args = {
        [1] = "StartQuest",
        [2] = _G.QUESTNAMEBOSS,
        [3] = _G.QUESTNUMBOSS
    }
    
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    wait(1.5)
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.PUKPOSBOSS
    end
else
    checkbossselect()
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.PUKPOSBOSS
    end
end



    
    
    
    
    
    
checkbossselect()
    
    for i,v in pairs(game.Workspace.Enemies:GetChildren()) do
    if v.Name == _G.NAMEMONBOSS and v.Humanoid.Health <= 0 then
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = _G.PUKPOSBOSS
    elseif v.Name == _G.NAMEMONBOSS  then
    v.HumanoidRootPart.Size = Vector3.new(30,30,30)
    v.HumanoidRootPart.Transparency = 0.8
    v.HumanoidRootPart.CanCollide = false
    v.Humanoid:ChangeState(11)
    v.Humanoid.WalkSpeed = 0
    game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0,15,15)
    require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework.CameraShaker).CameraShakeInstance.CameraShakeState = {FadingIn = 3,FadingOut =  2,Sustained = 0,Inactive = 1} 
    game:GetService'VirtualUser':CaptureController()
    game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
    require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework.CameraShaker).CameraShakeInstance.CameraShakeState = {FadingIn = 3,FadingOut =  2,Sustained = 0,Inactive = 1} 
    game:GetService'VirtualUser':CaptureController()
    game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
    require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework.CameraShaker).CameraShakeInstance.CameraShakeState = {FadingIn = 3,FadingOut =  2,Sustained = 0,Inactive = 1} 
    game:GetService'VirtualUser':CaptureController()
    game:GetService'VirtualUser':Button1Down(Vector2.new(1280, 672))
    end
    end
    
    end)
    end
    
    
    
    
    
            end)

    s1:addToggle("Melee", _G.UP_MELEE, function(value)
        _G.UP_MELEE = value
    end)
    
    s1:addToggle("Defense", false, function(value)
        _G.UP_DEFEN = value
    end)
    
    s1:addToggle("Sword", false, function(value)
        _G.UP_SWORD = value
    end)
    
    s1:addToggle("Gun", false, function(value)
        _G.UP_GUN = value
    end)
    
    s1:addToggle("Blox Fruit",_G.UP_BF, function(value)
        _G.UP_BF = value
    end)
    
    
    
    
    
    s1:addSlider("Select Stats", _G.AMOUNT, 1, 100, function(value)
    _G.AMOUNT = value
    end)
    
    
    -- MELEE
    spawn(function()
    while true do wait()
        if _G.UP_MELEE then
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AddPoint","Melee",_G.AMOUNT)
        end
    end
    end)
    -- Defense
    spawn(function()
    while true do wait()
        if _G.UP_DEFEN then
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AddPoint","Defense",_G.AMOUNT)
        end
    end
    end)
    -- Sword
    spawn(function()
    while true do wait()
        if _G.UP_SWORD then
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AddPoint","Sword",_G.AMOUNT)
        end
    end
    end)
    -- Gun
    spawn(function()
    while true do wait()
        if _G.UP_GUN then
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AddPoint","Gun",_G.AMOUNT)
        end
    end
    end)
    -- BF
    spawn(function()
    while true do wait()
        if _G.UP_BF then
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AddPoint","Demon Fruit",_G.AMOUNT)
        end
    end
    end)

local plr = {}
local gun = {}

for i, v in pairs(game.Players:GetChildren()) do
    table.insert(plr, v.Name)
end

lplr:addDropdown(
    "Select Player",
    plr,
    function(a)
        _G.PLAYERSELECT = a
    end
)

for i, v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
    table.insert(gun, v.Name)
end

lplr:addDropdown(
    "Selected Gun",
    gun,
    function(plys)
        _G.GUNSELECT = plys
    end
)

lplr:addButton(
    "Teleport to Player",
    function()
        if _G.PLAYERSELECT == nil then
            venyx:Notify("Error!", "Please, Select Player!")
        else
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                game.Players[_G.PLAYERSELECT].Character.HumanoidRootPart.CFrame
        end
    end
)

lplr:addButton(
    "Refresh Player",
    function()
        table.clear(plr)
        for i, v in pairs(game.Players:GetChildren()) do
            table.insert(plr, v.Name)
        end
    end
)

lplr:addButton(
    "Refresh Gun",
    function()
        table.clear(gun)
        for i, v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
            table.insert(gun, v.Name)
        end
    end
)

lplr:addToggle(
    "Spectate Player",
    nil,
    function(value)
        Sp = value
        local plr1 = game.Players.LocalPlayer.Character.Humanoid
        local plr2 = game.Players:FindFirstChild(_G.PLAYERSELECT)
        repeat
            wait(.1)
            game.Workspace.Camera.CameraSubject = plr2.Character.Humanoid
        until Sp == false
        game.Workspace.Camera.CameraSubject = game.Players.LocalPlayer.Character.Humanoid
    end
)

local KILL = nil

lplr:addToggle(
    "Kill Player [COMBAT]",
    nil,
    function(value)
        KILL = value
        if KILL == true then
            repeat
                autohaki()
                game.Players[_G.PLAYERSELECT].Character.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                    game.Players[_G.PLAYERSELECT].Character.HumanoidRootPart.CFrame * CFrame.new(0, 20, 5)
                game:GetService "VirtualUser":CaptureController()
                game:GetService "VirtualUser":Button1Down(Vector2.new(1280, 672))
                game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
                wait()
            until KILL == false
        else
            game.Players[_G.PLAYERSELECT].Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
            CheckLevel()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = QUESTPOS
        end
    end
)

spawn(
    function()
        pcall(
            function()
                game:GetService("RunService").Heartbeat:Connect(
                    function()
                        if _G.FAT then
                            Rig = require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework)
                            Cam =
                                require(
                                game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework.CameraShaker
                            )
                            Rig.activeController.timeToNextAttack = 1632600095.36
                            for i = 1, 5 do
                                Cam.CameraShakeInstance.CameraShakeState = {
                                    FadingIn = 3,
                                    FadingOut = 2,
                                    Sustained = 0,
                                    Inactive = 1
                                }
                                if KILL or _G.AUTOFARM_LEVEL or _G.AFB then
                                    Cam.CameraShakeInstance.CameraShakeState = {
                                        FadingIn = 3,
                                        FadingOut = 2,
                                        Sustained = 0,
                                        Inactive = 1
                                    }
                                    Rig.activeController.timeToNextAttack = 0
                                    game:GetService "VirtualUser":CaptureController()
                                    game:GetService "VirtualUser":Button1Down(Vector2.new(1280, 672))

                                    Cam.CameraShakeInstance.CameraShakeState = {
                                        FadingIn = 3,
                                        FadingOut = 2,
                                        Sustained = 0,
                                        Inactive = 1
                                    }
                                end
                            end
                        end
                    end
                )
            end
        )
    end
)

lplr:addToggle(
    "Kill Player [GUN]",
    nil,
    function(value)
        if _G.GUNSELECT == nil then
            venyx:Notify("Error!", "Please, Select Gun!")
        else
            _G.KILLGUN = value

            if _G.KILLGUN == true then
                repeat
                    autohaki()
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                        game.Players[_G.PLAYERSELECT].Character.HumanoidRootPart.CFrame * CFrame.new(0, 20, 5)
                    game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
                    if _G.KILLGUN and game.Players.LocalPlayer.Character:FindFirstChild(_G.GUNSELECT) then
                        local args = {
                            [1] = game:GetService("Players"):FindFirstChild(_G.PLAYERSELECT).Character.HumanoidRootPart.Position,
                            [2] = game:GetService("Players"):FindFirstChild(_G.PLAYERSELECT).Character.HumanoidRootPart
                        }
                        game:GetService("Players").LocalPlayer.Character[_G.GUNSELECT].RemoteFunctionShoot:InvokeServer(
                            unpack(args)
                        )
                        game:GetService "VirtualUser":CaptureController()
                        game:GetService "VirtualUser":Button1Down(Vector2.new(1280, 672))
                    end

                    wait()
                until _G.KILLGUN == false
            else
                game.Players[_G.PLAYERSELECT].Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
                CheckLevel()
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = QUESTPOS
            end
        end
    end
)

game:GetService("RunService").RenderStepped:connect(
    function()
        if _G.KILLGUN then
            game.Players.LocalPlayer.Character.Humanoid:ChangeState(11)
        end
    end
)

lplr:addToggle(
    "Aimbot Gun",
    nil,
    function(value)
        if _G.PLAYERSELECT == "" and value then
        else
            AimbotLock = value
        end
    end
)

local lp = game:GetService("Players").LocalPlayer
local mouse = lp:GetMouse()
mouse.Button1Down:Connect(
    function()
        if AimbotLock and game.Players.LocalPlayer.Character:FindFirstChild(_G.GUNSELECT) then
            local args = {
                [1] = game:GetService("Players"):FindFirstChild(_G.PLAYERSELECT).Character.HumanoidRootPart.Position,
                [2] = game:GetService("Players"):FindFirstChild(_G.PLAYERSELECT).Character.HumanoidRootPart
            }
            game:GetService("Players").LocalPlayer.Character[_G.GUNSELECT].RemoteFunctionShoot:InvokeServer(
                unpack(args)
            )
        end
    end
)

lplr:addToggle(
    "Aimbot Skill",
    nil,
    function(value)
        if _G.PLAYERSELECT == "lt_MOODz" then
            game.Players.LocalPlayer:kick("à¸ˆà¸°à¸†à¹ˆà¸²à¸à¸¹à¸«à¸£à¸­ :D")
        end
        AimbotSkill = value
        while AimbotSkill do
            wait()
            pcall(
                function()
                    if
                        game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool") and
                            game.Players.LocalPlayer.Character[
                                game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name
                            ]:FindFirstChild("MousePos")
                     then
                        local args = {
                            [1] = game:GetService("Players"):FindFirstChild(_G.PLAYERSELECT).Character.HumanoidRootPart.Position
                        }
                        game:GetService("Players").LocalPlayer.Character[
                            game.Players.LocalPlayer.Character:FindFirstChildOfClass("Tool").Name
                        ].RemoteEvent:FireServer(unpack(args))
                    end
                end
            )
        end
    end
)

lplr:addToggle(
    "Safe Zone",
    nil,
    function(value)
        _G.SGZ = value
        while _G.SGZ do
            wait()
            if _G.SGZ then
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                    CFrame.new(6018.955078125, 4735.9208984375, 7188.4248046875)
                wait()
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                    CFrame.new(7842.255859375, 4736.5561523438, 6097.0048828125)
                wait()
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                    CFrame.new(6442.53515625, 3637.3837890625, 6662.4252929688)
            else
                if game.PlaceId == 2753915549 then
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                        CFrame.new(CFrame.new(979.79895019531, 16.516613006592, 1429.0466308594))
                else
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                        CFrame.new(10.911778450012, 95.809577941895, 2774.4038085938)
                end
            end
        end
    end
)

local LocalPlayer = game:GetService "Players".LocalPlayer
local originalstam = LocalPlayer.Character.Energy.Value
function infinitestam()
    LocalPlayer.Character.Energy.Changed:connect(
        function()
            if InfinitsEnergy then
                LocalPlayer.Character.Energy.Value = originalstam
            end
        end
    )
end
spawn(
    function()
        while wait(.1) do
            if InfinitsEnergy then
                wait(0.3)
                originalstam = LocalPlayer.Character.Energy.Value
                infinitestam()
            end
        end
    end
)
nododgecool = false
function NoDodgeCool()
    if nododgecool then
        for i, v in next, getgc() do
            if game.Players.LocalPlayer.Character.Dodge then
                if typeof(v) == "function" and getfenv(v).script == game.Players.LocalPlayer.Character.Dodge then
                    for i2, v2 in next, getupvalues(v) do
                        if tostring(v2) == "0.4" then
                            repeat
                                wait(.1)
                                setupvalue(v, i2, 0)
                            until not nododgecool
                        end
                    end
                end
            end
        end
    end
end

llplr:addToggle(
    "Walk Speed",
    nil,
    function(Value)
        game.Players.LocalPlayer.Character.Movement.Disabled = Value
    end
)

llplr:addSlider(
    "Set Speed",
    16,
    16,
    400,
    function(value)
        game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = value
    end
)

llplr:addSlider(
    "Set Jump Power",
    50,
    50,
    400,
    function(value)
        game.Players.LocalPlayer.Character.Humanoid.JumpPower = value
    end
)

llplr:addToggle(
    "Dodge No Cooldown",
    nil,
    function(Value)
        nododgecool = Value
        NoDodgeCool()
    end
)

llplr:addToggle(
    "Infinity Stamina",
    nil,
    function(value)
        InfinitsEnergy = value
        originalstam = LocalPlayer.Character.Energy.Value
    end
)

ww:addButton(
    "Teleport To 1 World",
    function()
        local args = {
            [1] = "TravelMain"
        }
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    end
)

ww:addButton(
    "Teleport To 2 World",
    function()
        local args = {
            [1] = "TravelDressrosa"
        }
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    end
)

ww:addButton(
    "Teleport To 3 World",
    function()
        local args = {
            [1] = "TravelZou"
        }
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    end
)

ww:addButton(
    "Teleport SeaBeast",
    function()
        if game.PlaceId == 2753915549 then
        else
            pcall(
                function()
                    for i, v in pairs(game:GetService("Workspace").SeaBeasts:GetChildren()) do
                        if v:FindFirstChild("SeaBeast1") then
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                                v.SeaBeast1.HumanoidRootPart.CFrame
                        elseif v:FindFirstChild("SeaBeast2") then
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                                v.SeaBeast2.HumanoidRootPart.CFrame
                        elseif v:FindFirstChild("SeaBeast3") then
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                                v.SeaBeast3.HumanoidRootPart.CFrame
                        elseif v:FindFirstChild("SeaBeast4") then
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                                v.SeaBeast4.HumanoidRootPart.CFrame
                        end
                    end
                end
            )
        end
    end
)

if game.PlaceId == 2753915549 then
    tp:addDropdown(
        "Select Islands",
        {
            "WindMill",
            "Marine",
            "Middle Town",
            "Jungle",
            "Pirate Village",
            "Desert",
            "Snow Island",
            "MarineFord",
            "Colosseum",
            "Sky Island 1",
            "Sky Island 2",
            "Sky Island 3",
            "Prison",
            "Magma Village",
            "Under Water Island",
            "Fountain City",
            "Shank Room",
            "Mob Island"
        },
        function(Value)
            _G.TELEPORTISLAND = Value
        end
    )
elseif game.PlaceId == 4442272183 then
    tp:addDropdown(
        "Select Islands",
        {
            "cafe",
            "Frist Spot",
            "Dark Area",
            "Flamingo Mansion",
            "Flamingo Room",
            "Green Zone",
            "Factory",
            "Colossuim",
            "Zombie Island",
            "Two Snow Mountain",
            "Punk Hazard",
            "Cursed Ship",
            "Ice Castle",
            "Forgotten Island",
            "Ussop Island",
            "Mini Sky Island"
        },
        function(Value)
            _G.TELEPORTISLAND = Value
        end
    )
else
    tp:addDropdown(
        "Select Islands",
        {"Mansion", "Port Town", "Great Tree", "Castle On The Sea", "MiniSky", "Hydra Island", "Floating Turtle"},
        function(Value)
            _G.TELEPORTISLAND = Value
        end
    )
end

tp:addButton(
    "Teleport",
    function()
        if _G.TELEPORTISLAND == nil then
            spawn(
                function()
                    Notification.Notify("Failded !!", "Please, Choose You Location!!", "rbxassetid://4914902889")
                end
            )
        elseif _G.TELEPORTISLAND == "WindMill" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(979.79895019531, 16.516613006592, 1429.0466308594)
        elseif _G.TELEPORTISLAND == "Marine" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-2566.4296875, 6.8556680679321, 2045.2561035156)
        elseif _G.TELEPORTISLAND == "Middle Town" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-690.33081054688, 15.09425163269, 1582.2380371094)
        elseif _G.TELEPORTISLAND == "Jungle" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1612.7957763672, 36.852081298828, 149.12843322754)
        elseif _G.TELEPORTISLAND == "Pirate Village" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1181.3093261719, 4.7514905929565, 3803.5456542969)
        elseif _G.TELEPORTISLAND == "Desert" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(944.15789794922, 20.919729232788, 4373.3002929688)
        elseif _G.TELEPORTISLAND == "Snow Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(1347.8067626953, 104.66806030273, -1319.7370605469)
        elseif _G.TELEPORTISLAND == "MarineFord" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-4914.8212890625, 50.963626861572, 4281.0278320313)
        elseif _G.TELEPORTISLAND == "Colosseum" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1427.6203613281, 7.2881078720093, -2792.7722167969)
        elseif _G.TELEPORTISLAND == "Sky Island 1" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-4869.1025390625, 733.46051025391, -2667.0180664063)
        elseif _G.TELEPORTISLAND == "Sky Island 2" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-7874.0908203125, 5545.56640625, -370.306640625)
        elseif _G.TELEPORTISLAND == "Sky Island 3" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-7994.10546875, 5756.033203125, -1942.4979248047)
        elseif _G.TELEPORTISLAND == "Prison" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(4875.330078125, 5.6519818305969, 734.85021972656)
        elseif _G.TELEPORTISLAND == "Magma Village" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-5247.7163085938, 12.883934020996, 8504.96875)
        elseif _G.TELEPORTISLAND == "Under Water Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(3876.6374511719, 5.3731470108032, -1896.9306640625)
        elseif _G.TELEPORTISLAND == "Fountain City" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(5127.1284179688, 59.501365661621, 4105.4458007813)
        elseif _G.TELEPORTISLAND == "Shank Room" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1442.16553, 29.8788261, -28.3547478)
        elseif _G.TELEPORTISLAND == "Mob Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = 
                CFrame.new(-2850.20068, 7.39224768, 5354.99268)
        elseif _G.TELEPORTISLAND == "cafe" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-380.47927856445, 77.220390319824, 255.82550048828)
        elseif _G.TELEPORTISLAND == "Frist Spot" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-11.311455726624, 29.276733398438, 2771.5224609375)
        elseif _G.TELEPORTISLAND == "Dark Area" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(3780.0302734375, 22.652164459229, -3498.5859375)
        elseif _G.TELEPORTISLAND == "Flamingo Mansion" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-483.73370361328, 332.0383605957, 595.32708740234)
        elseif _G.TELEPORTISLAND == "Flamingo Room" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(2284.4140625, 15.152037620544, 875.72534179688)
        elseif _G.TELEPORTISLAND == "Green Zone" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-2448.5300292969, 73.016105651855, -3210.6306152344)
        elseif _G.TELEPORTISLAND == "Factory" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(424.12698364258, 211.16171264648, -427.54049682617)
        elseif _G.TELEPORTISLAND == "Colossuim" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1503.6224365234, 219.7956237793, 1369.3101806641)
        elseif _G.TELEPORTISLAND == "Zombie Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-5622.033203125, 492.19604492188, -781.78552246094)
        elseif _G.TELEPORTISLAND == "Two Snow Mountain" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(753.14288330078, 408.23559570313, -5274.6147460938)
        elseif _G.TELEPORTISLAND == "Punk Hazard" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-6127.654296875, 15.951762199402, -5040.2861328125)
        elseif _G.TELEPORTISLAND == "Cursed Ship" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(923.40197753906, 125.05712890625, 32885.875)
        elseif _G.TELEPORTISLAND == "Ice Castle" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(6148.4116210938, 294.38687133789, -6741.1166992188)
        elseif _G.TELEPORTISLAND == "Forgotten Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-3032.7641601563, 317.89672851563, -10075.373046875)
        elseif _G.TELEPORTISLAND == "Ussop Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(4816.8618164063, 8.4599885940552, 2863.8195800781)
        elseif _G.TELEPORTISLAND == "Mini Sky Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-288.74060058594, 49326.31640625, -35248.59375)
        elseif _G.TELEPORTISLAND == "Great Tree" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(2681.2736816406, 1682.8092041016, -7190.9853515625)
        elseif _G.TELEPORTISLAND == "Castle On The Sea" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-5044.7612304688, 314.85876464844, -2995.3803710938)
        elseif _G.TELEPORTISLAND == "MiniSky" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-260.65557861328, 49325.8046875, -35253.5703125)
        elseif _G.TELEPORTISLAND == "Port Town" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-294.20208740234, 29.756063461304, 5395.4111328125)
        elseif _G.TELEPORTISLAND == "Hydra Island" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(5228.8842773438, 604.23400878906, 345.0400390625)
        elseif _G.TELEPORTISLAND == "Floating Turtle" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-13274.528320313, 531.82073974609, -7579.22265625)
        elseif _G.TELEPORTISLAND == "Mansion" then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-12550.325195313, 337.51156616211, -7508.9936523438)
        end
    end
)
local tp = page3:addSection("Teleport Island 2")
if game.PlaceId == 2753915549 then
    tp:addButton(
        "WindMill",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(979.79895019531, 16.516613006592, 1429.0466308594)
        end
    )
    tp:addButton(
        "Marine",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-2566.4296875, 6.8556680679321, 2045.2561035156)
        end
    )
    tp:addButton(
        "Middle Town",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-690.33081054688, 15.09425163269, 1582.2380371094)
        end
    )
    tp:addButton(
        "Jungle",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1612.7957763672, 36.852081298828, 149.12843322754)
        end
    )
    tp:addButton(
        "Pirate Village",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1181.3093261719, 4.7514905929565, 3803.5456542969)
        end
    )
    tp:addButton(
        "Desert",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(944.15789794922, 20.919729232788, 4373.3002929688)
        end
    )
    tp:addButton(
        "Snow Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(1347.8067626953, 104.66806030273, -1319.7370605469)
        end
    )
    tp:addButton(
        "MarineFord",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-4914.8212890625, 50.963626861572, 4281.0278320313)
        end
    )
    tp:addButton(
        "Colosseum",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1427.6203613281, 7.2881078720093, -2792.7722167969)
        end
    )
    tp:addButton(
        "Sky Island 1",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-4869.1025390625, 733.46051025391, -2667.0180664063)
        end
    )
    tp:addButton(
        "Sky Island 2",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-7874.0908203125, 5545.56640625, -370.306640625)
        end
    )
    tp:addButton(
        "Sky Island 3",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-7994.10546875, 5756.033203125, -1942.4979248047)
        end
    )
    tp:addButton(
        "Prison",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(4875.330078125, 5.6519818305969, 734.85021972656)
        end
    )
    tp:addButton(
        "Magma Village",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-5247.7163085938, 12.883934020996, 8504.96875)
        end
    )
    tp:addButton(
        "Under Water Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(3876.6374511719, 5.3731470108032, -1896.9306640625)
        end
    )
    tp:addButton(
        "Fountain City",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(5127.1284179688, 59.501365661621, 4105.4458007813)
        end
    )
    tp:addButton(
        "Shank Room",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1442.16553, 29.8788261, -28.3547478)
        end
    )
    tp:addButton(
        "Mob Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-2850.20068, 7.39224768, 5354.99268)
        end
    )
elseif game.PlaceId == 4442272183 then
    tp:addButton(
        "cafe",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-380.47927856445, 77.220390319824, 255.82550048828)
        end
    )
    tp:addButton(
        "Frist Spot",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-11.311455726624, 29.276733398438, 2771.5224609375)
        end
    )
    tp:addButton(
        "Dark Area",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(3780.0302734375, 22.652164459229, -3498.5859375)
        end
    )
    tp:addButton(
        "Flamingo Mansion",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-483.73370361328, 332.0383605957, 595.32708740234)
        end
    )
    tp:addButton(
        "Flamingo Room",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(2284.4140625, 15.152037620544, 875.72534179688)
        end
    )
    tp:addButton(
        "Green Zone",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-2448.5300292969, 73.016105651855, -3210.6306152344)
        end
    )
    tp:addButton(
        "Factory",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(424.12698364258, 211.16171264648, -427.54049682617)
        end
    )
    tp:addButton(
        "Colossuim",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-1503.6224365234, 219.7956237793, 1369.3101806641)
        end
    )
    tp:addButton(
        "Zombie Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-5622.033203125, 492.19604492188, -781.78552246094)
        end
    )
    tp:addButton(
        "Two Snow Mountain",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(753.14288330078, 408.23559570313, -5274.6147460938)
        end
    )
    tp:addButton(
        "Punk Hazard",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-6127.654296875, 15.951762199402, -5040.2861328125)
        end
    )
    tp:addButton(
        "Cursed Ship",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(923.40197753906, 125.05712890625, 32885.875)
        end
    )
    tp:addButton(
        "Ice Castle",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(6148.4116210938, 294.38687133789, -6741.1166992188)
        end
    )
    tp:addButton(
        "Forgotten Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-3032.7641601563, 317.89672851563, -10075.373046875)
        end
    )
    tp:addButton(
        "Ussop Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(4816.8618164063, 8.4599885940552, 2863.8195800781)
        end
    )
    tp:addButton(
        "Mini Sky Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-288.74060058594, 49326.31640625, -35248.59375)
        end
    )
else
    tp:addButton(
        "Mansion",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-12550.325195313, 337.51156616211, -7508.9936523438)
        end
    )
    tp:addButton(
        "Port Town",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-294.20208740234, 29.756063461304, 5395.4111328125)
        end
    )

    tp:addButton(
        "Great Tree",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(2681.2736816406, 1682.8092041016, -7190.9853515625)
        end
    )

    tp:addButton(
        "Castle On The Sea",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-5044.7612304688, 314.85876464844, -2995.3803710938)
        end
    )

    tp:addButton(
        "MiniSky",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-260.65557861328, 49325.8046875, -35253.5703125)
        end
    )

    tp:addButton(
        "Hydra Island",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(5228.8842773438, 604.23400878906, 345.0400390625)
        end
    )

    tp:addButton(
        "Floating Turtle",
        function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame =
                CFrame.new(-13274.528320313, 531.82073974609, -7579.22265625)
        end
    )
end


raid:addToggle("Auto Kill Arua",false,function(value)
        RaidsArua = value
end)

spawn(function()
while wait(.1) do
   if RaidsArua  then
    for i,h in pairs(game.Workspace.Enemies:GetDescendants()) do
        if h.Name == "Humanoid"  then
            h.Parent.HumanoidRootPart.Size = Vector3.new(40,40,40)
            h.Parent.HumanoidRootPart.CanCollide = false
            h.Parent.Humanoid:ChangeState(11)
            sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", math.huge)
            h.Parent.Humanoid.Health = 0
            h.Parent.HumanoidRootPart.Size = Vector3.new(2,2,1)
            end
        end
        end
    end
end)

df2:addButton("Grab  All Devil Fruit", function()
    pcall(function()
                for i,v in pairs(game.Workspace:GetChildren()) do
                    if v:IsA "Tool" then
                        local i = {}

                            table.insert(i, game.Players.LocalPlayer.Character.HumanoidRootPart.Position)
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.Handle.CFrame * CFrame.new(0, 20, 0)
                            wait(0.5)
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.Handle.CFrame
                            wait()
                             game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(i[1])
                    end
                end
                end)
            end)
    
    
            df2:addToggle("Auto Grab  All Devil Fruit",nil,function(value)
                _G.AUTOBRINGFRUIT = value
                while _G.AUTOBRINGFRUIT do wait()
                    for i,v in pairs(game.Workspace:GetChildren()) do
                        if v:IsA "Tool" then
                            local i = {}

                             table.insert(i, game.Players.LocalPlayer.Character.HumanoidRootPart.Position)
                             game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.Handle.CFrame * CFrame.new(0, 20, 0)
                            wait(0.5)
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.Handle.CFrame
                            wait()
                             game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(i[1])
                        end
                    end
                end
            end)

local theme = venyx:addPage("Setting", 5012544693)
local colors = theme:addSection("Colors")
local gamee = theme:addSection("Game")
local themes = {
    Background = Color3.fromRGB(24, 24, 24),
    Glow = Color3.fromRGB(0, 0, 0),
    Accent = Color3.fromRGB(10, 10, 10),
    LightContrast = Color3.fromRGB(20, 20, 20),
    DarkContrast = Color3.fromRGB(14, 14, 14),
    TextColor = Color3.fromRGB(255, 255, 255)
}
gamee:addButton(
    "Rejoin Server",
    function()
        game:GetService("TeleportService"):Teleport(game.PlaceId, game.Players.LocalPlayer)
    end
)

gamee:addButton(
    "Server Hop",
    function()
        local PlaceID = game.PlaceId
        local AllIDs = {}
        local foundAnything = ""
        local actualHour = os.date("!*t").hour
        local Deleted = false
        local File =
            pcall(
            function()
                AllIDs = game:GetService("HttpService"):JSONDecode(readfile("NotSameServers.json"))
            end
        )
        if not File then
            table.insert(AllIDs, actualHour)
            writefile("NotSameServers.json", game:GetService("HttpService"):JSONEncode(AllIDs))
        end
        function TPReturner()
            local Site
            if foundAnything == "" then
                Site =
                    game.HttpService:JSONDecode(
                    game:HttpGet(
                        "https://games.roblox.com/v1/games/" .. PlaceID .. "/servers/Public?sortOrder=Asc&limit=100"
                    )
                )
            else
                Site =
                    game.HttpService:JSONDecode(
                    game:HttpGet(
                        "https://games.roblox.com/v1/games/" ..
                            PlaceID .. "/servers/Public?sortOrder=Asc&limit=100&cursor=" .. foundAnything
                    )
                )
            end
            local ID = ""
            if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
                foundAnything = Site.nextPageCursor
            end
            local num = 0
            for i, v in pairs(Site.data) do
                local Possible = true
                ID = tostring(v.id)
                if tonumber(v.maxPlayers) > tonumber(v.playing) then
                    for _, Existing in pairs(AllIDs) do
                        if num ~= 0 then
                            if ID == tostring(Existing) then
                                Possible = false
                            end
                        else
                            if tonumber(actualHour) ~= tonumber(Existing) then
                                local delFile =
                                    pcall(
                                    function()
                                        delfile("NotSameServers.json")
                                        AllIDs = {}
                                        table.insert(AllIDs, actualHour)
                                    end
                                )
                            end
                        end
                        num = num + 1
                    end
                    if Possible == true then
                        table.insert(AllIDs, ID)
                        wait()
                        pcall(
                            function()
                                writefile("NotSameServers.json", game:GetService("HttpService"):JSONEncode(AllIDs))
                                wait()
                                game:GetService("TeleportService"):TeleportToPlaceInstance(
                                    PlaceID,
                                    ID,
                                    game.Players.LocalPlayer
                                )
                            end
                        )
                        wait(4)
                    end
                end
            end
        end

        function Teleport()
            while wait() do
                pcall(
                    function()
                        TPReturner()
                        if foundAnything ~= "" then
                            TPReturner()
                        end
                    end
                )
            end
        end

        Teleport()
    end
)

for theme, color in pairs(themes) do
    colors:addColorPicker(
        theme,
        color,
        function(color3)
            venyx:setTheme(theme, color3)
        end
    )
end

venyx:SelectPage(venyx.pages[1], true)

-- SCRIPT

return library
