-- Load MaterialLua library
local Material = loadstring(game:HttpGet("https://raw.githubusercontent.com/Kinlei/MaterialLua/master/Module.lua"))()

-- Create the GUI
local GUI = Material.Load({
    Title = "🌌 Strong Ninja Simulator 🌌",
    Style = 2,
    SizeX = 320,
    SizeY = 320,
    Theme = "Dark",
    ColorOverrides = {
        MainFrame = Color3.fromRGB(0, 9, 55)
    }
})

-- Create a section in the GUI
local MainSection = GUI.New({
    Title = "Main"
})

-- Function to manage chams (ESP visual indicators)
local function manage_chams()
    local dwEntities = game:GetService("Players")
    local dwLocalPlayer = dwEntities.LocalPlayer 
    local dwRunService = game:GetService("RunService")

    local settings_tbl = {
        ESP_Enabled = true,
        ESP_TeamCheck = false,
        Chams = true,
        Chams_Color = Color3.fromRGB(0, 0, 255),
        Chams_Transparency = 0.1,
        Chams_Glow_Color = Color3.fromRGB(255, 0, 0)
    }

    function destroy_chams(char)
        for _, v in ipairs(char:GetChildren()) do
            if v:IsA("BasePart") and v.Transparency ~= 1 then
                local glow = v:FindFirstChild("Glow")
                local chams = v:FindFirstChild("Chams")
                if glow then glow:Destroy() end
                if chams then chams:Destroy() end
            end
        end
    end

    dwRunService.Heartbeat:Connect(function()
        if settings_tbl.ESP_Enabled then
            for _, v in ipairs(dwEntities:GetPlayers()) do
                if v ~= dwLocalPlayer then
                    local character = v.Character
                    if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
                        if not settings_tbl.ESP_TeamCheck or (settings_tbl.ESP_TeamCheck and v.Team ~= dwLocalPlayer.Team) then
                            for _, part in ipairs(character:GetChildren()) do
                                if part:IsA("BasePart") and part.Transparency ~= 1 then
                                    if settings_tbl.Chams then
                                        if not part:FindFirstChild("Glow") and not part:FindFirstChild("Chams") then
                                            local chams_box = Instance.new("BoxHandleAdornment", part)
                                            chams_box.Name = "Chams"
                                            chams_box.AlwaysOnTop = true
                                            chams_box.ZIndex = 4
                                            chams_box.Adornee = part
                                            chams_box.Color3 = settings_tbl.Chamas_Color
                                            chams_box.Transparency = settings_tbl.Chamas_Transparency
                                            chams_box.Size = part.Size + Vector3.new(0.02, 0.02, 0.02)

                                            local glow_box = Instance.new("BoxHandleAdornment", part)
                                            glow_box.Name = "Glow"
                                            glow_box.AlwaysOnTop = false
                                            glow_box.ZIndex = 3
                                            glow_box.Adornee = part
                                            glow_box.Color3 = settings_tbl.Chamas_Glow_Color
                                            glow_box.Size = chams_box.Size + Vector3.new(0.13, 0.13, 0.13)
                                        end
                                    else
                                        destroy_chams(character)
                                    end
                                end
                            end
                        else
                            destroy_chams(character)
                        end
                    else
                        destroy_chams(character)
                    end
                end
            end
        else
            for _, v in ipairs(dwEntities:GetPlayers()) do
                if v ~= dwLocalPlayer then
                    destroy_chams(v.Character)
                end
            end
        end
    end)
end

-- Toggle for Auto Swing
local AutoSwingToggle = MainSection.Toggle({
    Text = "Auto ESP Player",
    Callback = function(value)
        _G.Swing = value
        if _G.Swing then
            manage_chams() -- Start ESP functionality if Auto Swing is enabled
        else
            -- Disable ESP functionality
            game:GetService("RunService"):UnbindFromRenderStep("ESP_RenderStep")
            for _, player in ipairs(game.Players:GetPlayers()) do
                destroy_chams(player.Character)
            end
        end
    end,
    Enabled = false -- Start disabled by default
})

-- Function to toggle the ESP based on the Auto Swing toggle state
function toggle_esp(enabled)
    AutoSwingToggle:SetState(enabled)
end
