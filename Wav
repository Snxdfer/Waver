-- Initialization
local Stats = game:GetService('Stats')
local Players = game:GetService('Players')
local RunService = game:GetService('RunService')
local ReplicatedStorage = game:GetService('ReplicatedStorage')
local Workspace = game:GetService('Workspace')
local CoreGui = game:GetService('CoreGui')

local Nurysium_Util = loadstring(game:HttpGet('https://raw.githubusercontent.com/flezzpe/Nurysium/main/nurysium_helper.lua'))()
local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/flezzpe/Nurysium/main/nurysium_ui.lua"))()

local local_player = Players.LocalPlayer
local camera = Workspace.CurrentCamera

local nurysium_Data, hit_Sound, closest_Entity, parry_remote = nil, nil, nil, nil
local aura_table = {
    canParry = true,
    is_Spamming = false,
    parry_Range = 0,
    spam_Range = 0,
    hit_Count = 0,
    hit_Time = tick(),
    ball_Warping = tick(),
    is_ball_Warping = false
}

-- Settings
getgenv().aura_Enabled = false
getgenv().hit_sound_Enabled = false
getgenv().hit_effect_Enabled = false
getgenv().trail_Enabled = false
getgenv().self_effect_Enabled = false
getgenv().createSpamGui = false
getgenv().optimize_Enabled = false
getgenv().autoSpam_Enabled = false
getgenv().antiCurve_Enabled = false

-- UI Initialization
task.wait(1)
library:init("Waver⛅", game:GetService("UserInputService").TouchEnabled, CoreGui)
library:create_section("Combat", 17440545793)

-- Utility Functions
local function initialize_hit_sound(dataFolder_name)
    nurysium_Data = Instance.new('Folder', CoreGui)
    nurysium_Data.Name = dataFolder_name
    hit_Sound = Instance.new('Sound', nurysium_Data)
    hit_Sound.SoundId = 'rbxassetid://8632670510'
    hit_Sound.Volume = 5
end

local function get_closest_entity(object)
    local closest, max_distance = nil, math.huge
    for _, entity in pairs(Workspace.Alive:GetChildren()) do
        if entity:IsA('Model') and entity:FindFirstChild('HumanoidRootPart') and entity.Name ~= local_player.Name then
            local distance = (object.Position - entity.HumanoidRootPart.Position).Magnitude
            if distance < max_distance then
                closest_Entity = entity
                max_distance = distance
            end
        end
    end
    return closest_Entity
end

local function resolve_parry_remote()
    local services = {game:GetService('AdService'), game:GetService('SocialService')}
    for _, service in pairs(services) do
        local temp_remote = service:FindFirstChildOfClass('RemoteEvent')
        if temp_remote and temp_remote.Name:find('\n') then
            parry_remote = temp_remote
            break
        end
    end
end

-- Event Handlers
ReplicatedStorage.Remotes.ParrySuccess.OnClientEvent:Connect(function()
    if getgenv().hit_sound_Enabled then
        hit_Sound:Play()
    end
    if getgenv().hit_effect_Enabled then
        local hit_effect = game:GetObjects("rbxassetid://17407244385")[1]
        hit_effect.Parent = Nurysium_Util.getBall()
        hit_effect:Emit(4)
        task.delay(5, function() hit_effect:Destroy() end)
    end
end)

ReplicatedStorage.Remotes.ParrySuccessAll.OnClientEvent:Connect(function()
    aura_table.hit_Count += 1
    task.delay(0.15, function() aura_table.hit_Count -= 1 end)
end)

Workspace:WaitForChild("Balls").ChildRemoved:Connect(function()
    aura_table.hit_Count = 0
    aura_table.is_ball_Warping = false
    aura_table.is_Spamming = false
end)

-- Update Loop
task.spawn(function()
    RunService.PreRender:Connect(function()
        if not getgenv().aura_Enabled then return end

        if closest_Entity then
            local entity_root = Workspace.Alive:FindFirstChild(closest_Entity.Name)
            if entity_root and entity_root.Humanoid.Health > 0 then
                if aura_table.is_Spamming and local_player:DistanceFromCharacter(closest_Entity.HumanoidRootPart.Position) <= aura_table.spam_Range then
                    if parry_remote then
                        local target_position = closest_Entity.HumanoidRootPart.Position
                        if getgenv().antiCurve_Enabled then
                            target_position = camera.CFrame.Position + (target_position - camera.CFrame.Position).Unit * (local_player:DistanceFromCharacter(target_position) - 5)
                        end
                        parry_remote:FireServer(0.5, CFrame.new(camera.CFrame.Position, Vector3.zero), {[closest_Entity.Name] = target_position}, {target_position.X, target_position.Y}, false)
                    end
                end
            end
        end
    end)

    RunService.PreRender:Connect(function()
        if not getgenv().aura_Enabled then return end

        local ping = Stats.Network.ServerStatsItem['Data Ping']:GetValue() / 10
        local self = Nurysium_Util.getBall()
        if not self then return end

        self:GetAttributeChangedSignal('target'):Once(function() aura_table.canParry = true end)
        if self:GetAttribute('target') ~= local_player.Name or not aura_table.canParry then return end

        get_closest_entity(local_player.Character.PrimaryPart)

        local player_Position = local_player.Character.PrimaryPart.Position
        local ball_Position = self.Position
        local ball_Velocity = self.AssemblyLinearVelocity
        if self:FindFirstChild('zoomies') then
            ball_Velocity = self.zoomies.VectorVelocity
        end

        local ball_Direction = (player_Position - ball_Position).Unit
        local ball_Distance = local_player:DistanceFromCharacter(ball_Position)
        local ball_Dot = ball_Direction:Dot(ball_Velocity.Unit)
        local ball_Speed = ball_Velocity.Magnitude
        local ball_speed_Limited = math.min(ball_Speed / 1000, 0.1)
        local ball_predicted_Distance = (ball_Distance - ping / 15.5) - (ball_Speed / 3.5)

        local target_Position = closest_Entity.HumanoidRootPart.Position
        local target_Distance = local_player:DistanceFromCharacter(target_Position)
        local target_distance_Limited = math.min(target_Distance / 10000, 0.1)
        local target_Direction = (player_Position - closest_Entity.HumanoidRootPart.Position).Unit
        local target_Velocity = closest_Entity.HumanoidRootPart.AssemblyLinearVelocity
        local target_isMoving = target_Velocity.Magnitude > 0
        local target_Dot = target_isMoving and math.max(target_Direction:Dot(target_Velocity.Unit), 0)

        -- Improved Parry Prediction
        local parry_Distance = math.max(math.max(ping, 4) + ball_Speed / 3.5, 9.5)
        local predicted_Target_Position = target_Position + target_Velocity * (ball_Distance / ball_Speed)
        local parry_Position = camera.CFrame.Position + (ball_Direction * parry_Distance)

        aura_table.spam_Range = math.max(ping / 10, 15) + ball_Speed / 7
        aura_table.parry_Range = parry_Distance
        aura_table.is_Spamming = aura_table.hit_Count > 1 or ball_Distance < 13.5

        if ball_Dot < 0 then
            aura_table.ball_Warping = tick()
        end

        task.spawn(function()
            if (tick() - aura_table.ball_Warping) >= 0.25 + target_distance_Limited - ball_speed_Limited or ball_Distance <= 12 then
                aura_table.is_ball_Warping = false
                return
            end
            aura_table.is_ball_Warping = true
        end)

        if ball_Distance <= aura_table.parry_Range and not aura_table.is_Spamming and not aura_table.is_ball_Warping then
            if parry_remote then
                local target_position = predicted_Target_Position
                if getgenv().antiCurve_Enabled then
                    target_position = camera.CFrame.Position + (target_position - camera.CFrame.Position).Unit * (local_player:DistanceFromCharacter(target_position) - 5)
                end
               
