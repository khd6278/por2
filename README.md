--[[
    por穿墙 V4.0
    加载: loadstring(game:HttpGet("https://raw.githubusercontent.com/你的用户名/仓库名/main/por_hub.lua"))()
--]]

if getgenv().PorLoaded then return end
getgenv().PorLoaded = true

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PorNoclip"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- ==================== GUI 构建 ====================
local function createGUI()
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 220, 0, 190)
    mainFrame.Position = UDim2.new(0.1, 0, 0.1, 0)
    mainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    mainFrame.BorderColor3 = Color3.fromRGB(80, 0, 0)
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.Text = "por穿墙(4.0)"
    title.TextColor3 = Color3.fromRGB(255, 0, 0)
    title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    title.TextSize = 14
    title.Parent = mainFrame

    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 200, 0, 40)
    toggleBtn.Position = UDim2.new(0, 10, 0, 40)
    toggleBtn.Text = "穿墙: 关闭"
    toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
    toggleBtn.Parent = mainFrame

    local hideBtn = Instance.new("TextButton")
    hideBtn.Size = UDim2.new(0, 60, 0, 25)
    hideBtn.Position = UDim2.new(0, 10, 0, 90)
    hideBtn.Text = "隐藏"
    hideBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    hideBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    hideBtn.Parent = mainFrame

    local exitBtn = Instance.new("TextButton")
    exitBtn.Size = UDim2.new(0, 60, 0, 25)
    exitBtn.Position = UDim2.new(0, 80, 0, 90)
    exitBtn.Text = "退出"
    exitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    exitBtn.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
    exitBtn.Parent = mainFrame

    local destroyBtn = Instance.new("TextButton")
    destroyBtn.Size = UDim2.new(0, 200, 0, 30)
    destroyBtn.Position = UDim2.new(0, 10, 0, 125)
    destroyBtn.Text = "销毁"
    destroyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    destroyBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
    destroyBtn.TextSize = 14
    destroyBtn.Parent = mainFrame

    local miniBtn = Instance.new("TextButton")
    miniBtn.Size = UDim2.new(0, 50, 0, 50)
    miniBtn.Position = UDim2.new(0, 150, 0, 90)
    miniBtn.Text = "迷你"
    miniBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
    miniBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    miniBtn.Visible = false
    miniBtn.Active = true
    miniBtn.Draggable = true
    miniBtn.Parent = screenGui

    -- 确认弹窗
    local confirmFrame = Instance.new("Frame")
    confirmFrame.Size = UDim2.new(0, 200, 0, 100)
    confirmFrame.Position = UDim2.new(0.5, -100, 0.5, -50)
    confirmFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    confirmFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
    confirmFrame.BorderSizePixel = 2
    confirmFrame.Visible = false
    confirmFrame.Active = true
    confirmFrame.Draggable = true
    confirmFrame.Parent = screenGui

    local confirmText = Instance.new("TextLabel")
    confirmText.Size = UDim2.new(1, 0, 0, 40)
    confirmText.Position = UDim2.new(0, 0, 0, 10)
    confirmText.Text = "你确定吗？"
    confirmText.TextColor3 = Color3.fromRGB(255, 255, 255)
    confirmText.BackgroundTransparency = 1
    confirmText.TextSize = 16
    confirmText.Parent = confirmFrame

    local yesBtn = Instance.new("TextButton")
    yesBtn.Size = UDim2.new(0, 80, 0, 30)
    yesBtn.Position = UDim2.new(0, 15, 0, 55)
    yesBtn.Text = "确定"
    yesBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    yesBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
    yesBtn.Parent = confirmFrame

    local noBtn = Instance.new("TextButton")
    noBtn.Size = UDim2.new(0, 80, 0, 30)
    noBtn.Position = UDim2.new(0, 105, 0, 55)
    noBtn.Text = "取消"
    noBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    noBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    noBtn.Parent = confirmFrame

    return mainFrame, toggleBtn, hideBtn, exitBtn, destroyBtn, miniBtn, confirmFrame, yesBtn, noBtn
end

local mainFrame, toggleBtn, hideBtn, exitBtn, destroyBtn, miniBtn, confirmFrame, yesBtn, noBtn = createGUI()

-- ==================== 状态管理 ====================
local noclip = false
local connections = {}
local originalCollisions = {}
local isCleanedUp = false
local pendingAction = nil

local function getCharacterParts(character)
    local parts = {}
    for _, descendant in pairs(character:GetDescendants()) do
        if descendant:IsA("BasePart") then
            table.insert(parts, descendant)
        end
    end
    return parts
end

local function enableNoclip(character)
    local parts = getCharacterParts(character)
    for _, part in pairs(parts) do
        if originalCollisions[part] == nil then
            originalCollisions[part] = part.CanCollide
        end
        part.CanCollide = false
    end
end

local function disableNoclip(character)
    for part, originalState in pairs(originalCollisions) do
        if part and part.Parent then
            part.CanCollide = originalState
        end
    end
    table.clear(originalCollisions)
end

local function updateUI(state)
    noclip = state
    if noclip then
        toggleBtn.Text = "穿墙: 开启"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
    else
        toggleBtn.Text = "穿墙: 关闭"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
    end
end

local function toggleNoclip(forcedState)
    local newState = (forcedState ~= nil) and forcedState or not noclip
    if newState == noclip then return end

    updateUI(newState)

    local character = player.Character
    if character then
        if newState then
            enableNoclip(character)
        else
            disableNoclip(character)
        end
    end
end

-- 确认弹窗逻辑
local function showConfirm(action)
    pendingAction = action
    confirmFrame.Visible = true
    mainFrame.Visible = false
    miniBtn.Visible = false
end

local function hideConfirm()
    pendingAction = nil
    confirmFrame.Visible = false
    mainFrame.Visible = true
end

-- 清理函数
local function cleanup()
    if isCleanedUp then return end
    isCleanedUp = true

    if noclip and player.Character then
        disableNoclip(player.Character)
    end

    for _, conn in pairs(connections) do
        conn:Disconnect()
    end
    connections = {}

    if screenGui and screenGui.Parent then
        screenGui:Destroy()
    end
end

-- ==================== 事件连接 ====================
local function onCharacterAdded(character)
    if noclip and not isCleanedUp then
        enableNoclip(character)
    end
end

local function onCharacterRemoving(character)
    if noclip and not isCleanedUp then
        disableNoclip(character)
    end
end

if player.Character then
    onCharacterAdded(player.Character)
end
table.insert(connections, player.CharacterAdded:Connect(onCharacterAdded))
table.insert(connections, player.CharacterRemoving:Connect(onCharacterRemoving))

local function onDescendantAdded(descendant)
    if noclip and not isCleanedUp and descendant:IsA("BasePart") then
        if originalCollisions[descendant] == nil then
            originalCollisions[descendant] = descendant.CanCollide
        end
        descendant.CanCollide = false
    end
end

if player.Character then
    table.insert(connections, player.Character.DescendantAdded:Connect(onDescendantAdded))
end
table.insert(connections, player.CharacterAdded:Connect(function(character)
    if isCleanedUp then return end
    table.insert(connections, character.DescendantAdded:Connect(onDescendantAdded))
end))

-- GUI 按钮事件
table.insert(connections, toggleBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    toggleNoclip()
end))

table.insert(connections, hideBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    mainFrame.Visible = false
    miniBtn.Visible = true
    miniBtn.Text = "por"
end))

table.insert(connections, miniBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    mainFrame.Visible = true
    miniBtn.Visible = false
    miniBtn.Text = "迷你"
end))

table.insert(connections, exitBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    showConfirm("exit")
end))

table.insert(connections, destroyBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    showConfirm("destroy")
end))

table.insert(connections, yesBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    cleanup()
end))

table.insert(connections, noBtn.MouseButton1Click:Connect(function()
    if isCleanedUp then return end
    hideConfirm()
end))

table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed or isCleanedUp then return end
    if input.KeyCode == Enum.KeyCode.F then
        toggleNoclip()
    end
end))
