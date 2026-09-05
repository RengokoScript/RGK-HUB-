

-- RGK HUB - Steal an Egg (Strict Max 2 Players Only)
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Remove a interface anterior se já existir
if PlayerGui:FindFirstChild("RGKHub") then
    PlayerGui.RGKHub:Destroy()
end

-- Criação da Interface Gráfica
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RGKHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 260, 0, 165)
MainFrame.Position = UDim2.new(0.5, -130, 0.5, -82)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = "RGK HUB"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

local HopButton = Instance.new("TextButton")
HopButton.Size = UDim2.new(0.85, 0, 0, 42)
HopButton.Position = UDim2.new(0.075, 0, 0.30, 0)
HopButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
HopButton.Text = "Server Hop (Max 2 Players)"
HopButton.TextColor3 = Color3.fromRGB(255, 255, 255)
HopButton.TextSize = 14
HopButton.Font = Enum.Font.SourceSansBold
HopButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 6)
ButtonCorner.Parent = HopButton

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, 0, 0, 22)
StatusLabel.Position = UDim2.new(0, 0, 0.62, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Status: Pronto"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.TextSize = 12
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.Parent = MainFrame

local CreditLabel = Instance.new("TextLabel")
CreditLabel.Size = UDim2.new(1, -15, 0, 18)
CreditLabel.Position = UDim2.new(0, 0, 0.82, 0)
CreditLabel.BackgroundTransparency = 1
CreditLabel.Text = "By Rengoko Script"
CreditLabel.TextColor3 = Color3.fromRGB(100, 100, 100)
CreditLabel.TextSize = 10
CreditLabel.Font = Enum.Font.SourceSansItalic
CreditLabel.TextXAlignment = Enum.TextXAlignment.Right
CreditLabel.Parent = MainFrame

local isHopping = false

local function ServerHop()
    if isHopping then return end
    isHopping = true
    
    task.spawn(function()
        local placeId = game.PlaceId
        local targetServer = nil
        
        while isHopping and not targetServer do
            StatusLabel.Text = "Status: Buscando servidor (max 2)..."
            local cursor = ""
            local validServers = {}
            
            repeat
                local url = "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"
                if cursor ~= "" then
                    url = url .. "&cursor=" .. cursor
                end
                
                local success, response = pcall(function()
                    return game:HttpGet(url)
                end)
                
                if success and response then
                    local decodeSuccess, data = pcall(function()
                        return HttpService:JSONDecode(response)
                    end)
                    
                    if decodeSuccess and data and data.data then
                        for _, server in ipairs(data.data) do
                            if server.id ~= game.JobId and server.playing and server.playing <= 2 then
                                table.insert(validServers, server)
                            end
                        end
                        
                        if #validServers > 0 or not data.nextPageCursor then
                            break
                        end
                        cursor = data.nextPageCursor
                    else
                        break
                    end
                else
                    break
                end
                task.wait(0.1)
            until not cursor or cursor == ""
            
            if #validServers > 0 then
                table.sort(validServers, function(a, b)
                    return a.playing < b.playing
                end)
                
                for _, target in ipairs(validServers) do
                    StatusLabel.Text = "Status: Entrando (" .. target.playing .. " players)..."
                    
                    local tpSuccess = pcall(function()
                        TeleportService:TeleportToPlaceInstance(placeId, target.id, LocalPlayer)
                    end)
                    
                    if tpSuccess then
                        targetServer = target.id
                        task.wait(6)
                        break
                    else
                        task.wait(0.3)
                    end
                end
            end
            
            if not targetServer then
                StatusLabel.Text = "Status: Nenhum <=2 encontrado. Retentando..."
                task.wait(2)
            end
        end
        
        isHopping = false
    end)
end

HopButton.MouseButton1Click:Connect(ServerHop)
