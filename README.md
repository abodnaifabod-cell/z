loadstring([[
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local LocalPlayer = Players.LocalPlayer

local function setTargetDevice(device)
    if writefile then pcall(writefile, "sys_cfg_v5_cache.bin", device) end
    _G.TargetDevice = device
end

local function getTargetDevice()
    return _G.TargetDevice or "Pc"
end

local function applyDeviceUiFixes()
    local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return end
    local screen = playerGui:FindFirstChild("Screen")
    if screen then screen.Enabled = true end
end

local function spoofDeviceInGc(device)
    if not device or not getgc then return end
    for _, obj in ipairs(getgc(true)) do
        if type(obj) == "table" then
            pcall(function()
                if rawget(obj, "Device") then rawset(obj, "Device", device) end
            end)
        end
    end
end

local function main()
    if not game:IsLoaded() then game.Loaded:Wait() end
    
    local function reapply()
        pcall(applyDeviceUiFixes)
        pcall(spoofDeviceInGc, getTargetDevice())
    end

    local gui = Instance.new("ScreenGui")
    gui.Name = "B_Spoofer"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.Parent = gethui and gethui() or LocalPlayer:WaitForChild("PlayerGui")

    local btn = Instance.new("TextButton", gui)
    btn.Size = UDim2.fromOffset(36, 36)
    btn.Position = UDim2.new(0, 16, 0.5, -18)
    btn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    btn.Text = "B"
    btn.TextColor3 = Color3.fromRGB(0, 0, 0)
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamBold
    btn.Draggable = true
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 7)

    local panel = Instance.new("Frame", gui)
    panel.Size = UDim2.fromOffset(250, 180)
    panel.Position = UDim2.new(0.5, -125, 0.5, -90)
    panel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    panel.Visible = false
    Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 12)

    local stroke = Instance.new("UIStroke", panel)
    stroke.Color = Color3.fromRGB(255, 255, 255)
    stroke.Transparency = 0.5

    local title = Instance.new("TextLabel", panel)
    title.Size = UDim2.new(1, -20, 0, 40)
    title.Position = UDim2.fromOffset(14, 0)
    title.BackgroundTransparency = 1
    title.Text = "B"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 13
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left

    local list = Instance.new("UIListLayout", panel)
    list.Padding = UDim.new(0, 6)
    list.SortOrder = Enum.SortOrder.LayoutOrder

    local platforms = {"Console", "Mobile", "Pc"}
    for i, name in ipairs(platforms) do
        local pBtn = Instance.new("TextButton", panel)
        pBtn.Size = UDim2.new(1, -28, 0, 38)
        pBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
        pBtn.Text = name
        pBtn.TextColor3 = Color3.fromRGB(210, 215, 225)
        pBtn.TextSize = 11
        pBtn.Font = Enum.Font.GothamMedium
        pBtn.LayoutOrder = i
        Instance.new("UICorner", pBtn).CornerRadius = UDim.new(0, 8)

        pBtn.MouseButton1Click:Connect(function()
            setTargetDevice(name)
            if queue_on_teleport then
                queue_on_teleport("_G.TargetDevice = '" .. name .. "'")
            end
            TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
        end)
    end

    btn.MouseButton1Click:Connect(function()
        panel.Visible = not panel.Visible
    end)

    task.spawn(function()
        while true do
            reapply()
            task.wait(1)
        end
    end)
end

main()
]])()