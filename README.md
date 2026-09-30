-- shako.win | Rivals HVH v5.32.14c · Obsidian · lagfix · spoof tab
-- Avatar mundo + crash fixes · stable boot · no ESP reconnect leak

-- Boot: 1 sola instancia. Si FALLA el load, se puede reintentar.
-- Si ya cargó bien, ejecutar otra vez NO abre segunda UI.
pcall(function()
    if not game:IsLoaded() then game.Loaded:Wait() end
end)

if getgenv().antiSleepy_loaded then
    -- ya hay una instancia activa → no abrir otra
    return
end
getgenv().antiSleepy_loaded = true

-- ==================== SOLO RIVALS ====================
-- GameId oficial de Rivals
local RIVALS_GAME_ID = 6035872082
local function notSupportedKick()
    local msg = "this game not supported"
    warn("[shako.win] " .. msg .. " | GameId=" .. tostring(game.GameId))
    pcall(function()
        local lp = game:GetService("Players").LocalPlayer
        if lp then lp:Kick(msg) end
    end)
    -- si el kick no corre, forzar leave
    pcall(function()
        game:GetService("TeleportService"):Teleport(0) -- invalido = force exit intent
    end)
    getgenv().antiSleepy_loaded = false
end

if tonumber(game.GameId) ~= RIVALS_GAME_ID then
    notSupportedKick()
    return
end
-- =====================================================

local function shakoFail(msg)
    warn("[shako]", msg)
    getgenv().antiSleepy_loaded = false -- permite re-ejecutar SOLO si falló
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
    for _ = 1, 100 do
        LocalPlayer = Players.LocalPlayer
        if LocalPlayer then break end
        task.wait(0.05)
    end
end
if not LocalPlayer then
    shakoFail("LocalPlayer nil — reintenta en 1s")
    return
end

-- PlayerGui listo (sin esto la UI a veces no sale al entrar)
pcall(function() LocalPlayer:WaitForChild("PlayerGui", 12) end)

local Camera = workspace.CurrentCamera

-- ==================== Obsidian UI ====================
local repo = "https://raw.githubusercontent.com/deividcomsono/Obsidian/main/"
local Library, ThemeManager, SaveManager

local function httpGet(url)
    local ok, body = pcall(function()
        return game:HttpGet(url)
    end)
    if ok and body and #body > 100 then return body end
    return nil
end

local function try_load(url)
    local body = httpGet(url)
    if not body then return nil end
    local ok, res = pcall(function() return loadstring(body)() end)
    if ok and res then return res end
    return nil
end

-- Obsidian first (única UI)
Library = try_load(repo .. "Library.lua")
ThemeManager = try_load(repo .. "addons/ThemeManager.lua")
SaveManager = try_load(repo .. "addons/SaveManager.lua")

-- retry once si falló
if not Library then
    task.wait(0.4)
    Library = try_load(repo .. "Library.lua")
end
if not ThemeManager then
    ThemeManager = try_load(repo .. "addons/ThemeManager.lua")
end
if not SaveManager then
    SaveManager = try_load(repo .. "addons/SaveManager.lua")
end

if not Library or type(Library.CreateWindow) ~= "function" then
    shakoFail("Obsidian no cargó — revisa internet / executor")
    return
end

pcall(function() Library.ShowCustomCursor = true end)

local function Icon(n)
    return nil
end

if ThemeManager and ThemeManager.SetLibrary then
    pcall(function()
        ThemeManager:SetLibrary(Library)
        ThemeManager:SetFolder("shako_win")
    end)
end
if SaveManager and SaveManager.SetLibrary then
    pcall(function()
        SaveManager:SetLibrary(Library)
        SaveManager:SetFolder("shako_win/configs")
        if ThemeManager and SaveManager.IgnoreThemeSettings then
            pcall(function() SaveManager:IgnoreThemeSettings() end)
        end
        pcall(function() SaveManager:SetIgnoreIndexes({"MenuKeybind"}) end)
    end)
end

local Window
do
    local ok, win = pcall(function()
        -- ventana mas ancha + fuente limpia (no Code)
        pcall(function()
            Library.Font = Enum.Font.Gotham
            if Library.SetFontSize then
                Library:SetFontSize(13)
            else
                Library.FontSize = 13
            end
        end)
        return Library:CreateWindow({
            Title = "shako.win  |  Rivals",
            Footer = "v5.32.14c · Obsidian",
            Center = true,
            AutoShow = true,
            Resizable = true,
            NotifySide = "Right",
            ShowCustomCursor = true,
        })
    end)
    if not ok or not win then
        shakoFail("CreateWindow falló — vuelve a ejecutar")
        return
    end
    Window = win
end

-- Forzar label Rivals (no Marketplace)
pcall(function()
    if Library.SetWatermark then
        Library:SetWatermark("shako.win · Rivals")
    end
    if Library.Watermark then
        pcall(function() Library.Watermark.Text = "shako.win · Rivals" end)
    end
end)

-- silenciar notificaciones de la UI (menos Config)
pcall(function()
    if Library and Library.Notify then
        getgenv()._ShakoLibraryNotify = Library.Notify
        Library.Notify = function(self, data)
            -- bloquear spam de toggles; configs usan Notify() → StarterGui
            return
        end
    end
end)

function Notify(title, text)
    if tostring(title) ~= "Config" then return end
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Config",
            Text = tostring(text or ""),
            Duration = 3,
        })
    end)
    pcall(function()
        local fn = getgenv()._ShakoLibraryNotify
        if type(fn) == "function" then
            fn(Library, { Title = "Config", Description = tostring(text or ""), Time = 3 })
        end
    end)
end
getgenv()._ShakoBootAt = tick()


-- ==================== AURORA SPOOFERS (scoped, no AC bypass) ====================
local _srcOk, _srcErr = pcall(function()

-- AURORA SOURCE SPOOFERS (scoped)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ReplicatedFirst = game:GetService("ReplicatedFirst")
local CoreGui = game:GetService("CoreGui")
local MarketplaceService = game:GetService("MarketplaceService")
local SoundService = game:GetService("SoundService")
local Workspace = workspace
local Lighting = game:GetService("Lighting")

local LocalChar = LocalPlayer.Character
pcall(function()
    if LocalChar then
        LocalChar:WaitForChild("Humanoid", 3)
        LocalChar:WaitForChild("HumanoidRootPart", 3)
    end
end)
LocalPlayer.CharacterAdded:Connect(function(c)
    LocalChar = c
end)

local SpoofConnections = {}
local function Bind(name, signal, fn)
    if SpoofConnections[name] then pcall(function() SpoofConnections[name]:Disconnect() end) end
    SpoofConnections[name] = signal:Connect(fn)
end
local function Unbind(name)
    if SpoofConnections[name] then pcall(function() SpoofConnections[name]:Disconnect() end) SpoofConnections[name] = nil end
end

local modules = {}
pcall(function()
    local PS = LocalPlayer:WaitForChild("PlayerScripts", 10)
    local Controllers = PS and PS:FindFirstChild("Controllers")
    local RSModules = ReplicatedStorage:FindFirstChild("Modules")

    if Controllers then
        pcall(function() modules.PlayerDataController = require(Controllers:WaitForChild("PlayerDataController", 5)) end)
        pcall(function() modules.LeaderboardController = require(Controllers:WaitForChild("LeaderboardController", 5)) end)
        pcall(function() modules.ControlsController = require(Controllers:WaitForChild("ControlsController", 5)) end)
        pcall(function() modules.MonetizationController = require(Controllers:WaitForChild("MonetizationController", 5)) end)
    end
    if RSModules then
        pcall(function() modules.SeasonLibrary = require(RSModules:WaitForChild("SeasonLibrary", 5)) end)
        pcall(function() modules.MonetizationLibrary = require(RSModules:WaitForChild("MonetizationLibrary", 5)) end)
        pcall(function() modules.CosmeticLibrary = require(RSModules:WaitForChild("CosmeticLibrary", 5)) end)
        pcall(function() modules.ShopLibrary = require(RSModules:WaitForChild("ShopLibrary", 5)) end)
        pcall(function() modules.ItemLibrary = require(RSModules:WaitForChild("ItemLibrary", 5)) end)
    end
end)

task.spawn(function()
    pcall(function()
        if modules.PlayerDataController and modules.PlayerDataController.WaitUntilLoaded then
            modules.PlayerDataController:WaitUntilLoaded()
        end
    end)
end)

-- Connections -> SpoofConnections
local function Bind(name, signal, fn)
    if SpoofConnections[name] then SpoofConnections[name]:Disconnect() end
    SpoofConnections[name] = signal:Connect(fn)
end
local function Unbind(name)
    if SpoofConnections[name] then SpoofConnections[name]:Disconnect(); SpoofConnections[name] = nil end
end

local function SafeGet(key, fallback)
    local ok, v = pcall(function()
        return modules.PlayerDataController and modules.PlayerDataController:Get(key)
    end)
    return (ok and v ~= nil) and v or fallback
end

local states = {
    spoofer_state = {
        device = "Desktop",
        old_device = tostring(modules.ControlsController and modules.ControlsController.CurrentControls or "Desktop"),
        spoof_device = false,
        device_spam = false,
        spam_rate = 1,

        display_name_value = LocalPlayer.DisplayName,
        username_value = LocalPlayer.Name,
        display_name = false,
        username = false,
        name_spoof_conn = {},

        avatar_userid = LocalPlayer.UserId,
        spoof_avatar = false,
        thumb_spoof_conn = {},

        anonymous_mode = false,
        fake_names = {},
        anon_connections = {},

        leaderboard = {
            elo_value = 0,
            streak_value = LocalPlayer:GetAttribute("StatisticDuelsWinStreak") or 0,
            kills_value = 0,
            wins_value = 0,
            level_value = LocalPlayer:GetAttribute("Level") or 0,
            old_streak_value = LocalPlayer:GetAttribute("StatisticDuelsWinStreak"),
            old_level_value = LocalPlayer:GetAttribute("Level"),
            ELO = false, Streak = false, Kills = false, Wins = false, Level = false,
        },

        badges = {Premium = false, Verified = false, Influencer = false, Admin = false},

        charm = {
            charm_rank = "Archnemesis", arch_rank = 1,
            s0_charm = true, s1_charm = true, s2_charm = true, s3_charm = true,
        },

        currency = {
            weapon_keys = false, unlock_tokens = false, event_currency = false,
            glory = false, skin_tickets = false,
            weapon_keys_value = SafeGet("WeaponKeys", 0),
            unlock_tokens_value = SafeGet("UnlockTokens", 0),
            event_currency_value = SafeGet("EventCurrency", 0),
            glory_value = SafeGet("Glory", 0),
            skin_tickets_value = SafeGet("SkinTickets", 0),
        },

        duel_history = {
            duel_history_value = SafeGet("DuelHistory", {}),
            logged_elo_events = SafeGet("LoggedELOEvents", {}),
            match_index = 1, mode = "Ranked", result = "Victory", map = "Factory",
            team1score = 5, team2score = 0, dueler_index = 1,
            username = "", display_name = "",
            kills = 15, deaths = 2, assists = 6, damage = 5000, elo = 2000, elo_change = 35,
        },

        marketplace = {
            fake_bundles_owned = SafeGet("GamepassBundlesClaimed", {}),
            fake_gift_robux_spent = SafeGet("GiftRobuxSpentProgress", 0),
            fake_gift_rewards_claimed = SafeGet("GiftRobuxSpentRewardsClaimed", {}),
            fake_robux = false, robux_amount = 0, fake_gifting = false,
        },
    },

    skinchanger_state = {
        fake_owned = SafeGet("CosmeticInventory", {}),
        fake_weapon_owned = SafeGet("WeaponInventory", {}),
        equipped = {},
        favorites = {},
        fake_weapon_names = {},
        constructing_weapon = nil,
        last_used_weapon = nil,
        placed_object_map = {},
        viewing_profile = nil,

        general_unlocker = {
            unlock_type = "Skin",
            unlock_rarity = "Common",
        },

        specific_unlocker = {
            unlock_type = "Skin",
            cosmetic_name = "",
            weapon_name = "",
        },

        equip_unlocker = {
            unlock_type = "Skin",
            cosmetic_name = "",
            weapon_name = "",
            inverted = false,
        },
    },

    inventory_state = {
        fake_owned = SafeGet("UnclaimedRewards", {}),
    },
}

local kCharmEloMap = {
    ["Unranked"] = -1, ["Bronze 1"] = 0, ["Bronze 2"] = 200, ["Bronze 3"] = 400,
    ["Silver 1"] = 600, ["Silver 2"] = 800, ["Silver 3"] = 1000,
    ["Gold 1"] = 1200, ["Gold 2"] = 1400, ["Gold 3"] = 1600,
    ["Platinum 1"] = 1800, ["Platinum 2"] = 2000, ["Platinum 3"] = 2200,
    ["Diamond 1"] = 2400, ["Diamond 2"] = 2600, ["Diamond 3"] = 2800,
    ["Onyx 1"] = 3000, ["Onyx 2"] = 3200, ["Onyx 3"] = 3400,
    ["Nemesis"] = 3600, ["Archnemesis"] = 3600,
}
local kSeasonNameMap = {[0]="Zero",[1]="Warp",[2]="Polar",[3]="Fame", Zero=0, Warp=1, Polar=2, Fame=3}
local kSeasonToggleMap = {[0]="s0_charm",[1]="s1_charm",[2]="s2_charm",[3]="s3_charm"}
getgenv()._ShakoCharmEloMap = kCharmEloMap
getgenv()._ShakoSeasonNameMap = kSeasonNameMap
getgenv()._ShakoSeasonToggleMap = kSeasonToggleMap
local kWordList = {
    "late","couple","economic","obscure","landscape","spot","college","rainbow","leg","mixture",
    "frank","commitment","draft","motivation","fuss","settle","jelly","confine","pin","modernize",
    "velvet","quarter","physical","differ","dish","aisle","happen","council","appeal","depart",
    "urgency","vessel",
}

pcall(function()
    if not modules.PlayerDataController then return end
    local original = modules.PlayerDataController.Get
    modules.PlayerDataController.Get = function(self, key, ...)
        local data = original(self, key, ...)
        local s = states.spoofer_state
        if key == "WeaponKeys" and s.currency.weapon_keys then return s.currency.weapon_keys_value end
        if key == "UnlockTokens" and s.currency.unlock_tokens then return s.currency.unlock_tokens_value end
        if key == "EventCurrency" and s.currency.event_currency then return s.currency.event_currency_value end
        if key == "Glory" and s.currency.glory then return s.currency.glory_value end
        if key == "SkinTickets" and s.currency.skin_tickets then return s.currency.skin_tickets_value end
        if key == "DuelHistory" then return s.duel_history.duel_history_value or data end
        if key == "LoggedELOEvents" then return s.duel_history.logged_elo_events or data end
        if key == "GamepassBundlesClaimed" then return s.marketplace.fake_bundles_owned or data end
        if key == "GiftRobuxSpentProgress" then return s.marketplace.fake_gift_robux_spent or data end
        if key == "GiftRobuxSpentRewardsClaimed" then return s.marketplace.fake_gift_rewards_claimed or data end
        if key == "CosmeticInventory" then return states.skinchanger_state.fake_owned or data end
        if key == "WeaponInventory" or key == "FreeWeaponUnlockCheck" then return states.skinchanger_state.fake_weapon_owned or data end
        if key == "UnclaimedRewards" then return states.inventory_state.fake_owned or data end
        return data
    end
end)

pcall(function()
    if not modules.CosmeticLibrary then return end
    local cos = modules.CosmeticLibrary

    local function WrapOwnership(old, expected_args)
        return function(...)
            local args = {...}
            local name = args[expected_args]
            if type(name) == "string" and states.skinchanger_state.fake_owned[name] then return true end
            if old then return old(...) end
            return false
        end
    end

    local orig_normal = cos.OwnsCosmeticNormally
    cos.OwnsCosmeticNormally = WrapOwnership(orig_normal, 3)
    local orig_universal = cos.OwnsCosmeticUniversally
    cos.OwnsCosmeticUniversally = WrapOwnership(orig_universal, 3)
    local orig_for_something = cos.OwnsCosmeticForSomething
    cos.OwnsCosmeticForSomething = WrapOwnership(orig_for_something, 3)
    local orig_for_weapon = cos.OwnsCosmeticForWeapon
    cos.OwnsCosmeticForWeapon = WrapOwnership(orig_for_weapon, 3)

    local orig_owns = cos.OwnsCosmetic
    cos.OwnsCosmetic = function(self, inventory, name, weapon)
        if type(name) == "string" and states.skinchanger_state.fake_owned[name] then return true end
        if orig_owns then return orig_owns(self, inventory, name, weapon) end
        return false
    end
end)

pcall(function()
    if not modules.PlayerDataController then return end
    local pdc = modules.PlayerDataController

    if pdc.HasGamepass then
        local orig_hasgamepass = pdc.HasGamepass
        pdc.HasGamepass = function(p11, ...)
            local has_gamepass = orig_hasgamepass and orig_hasgamepass(p11, ...)
            local name = ({...})[1]
            return type(name) == "string" and states.spoofer_state.marketplace.fake_bundles_owned[name] or has_gamepass
        end
    end

    if pdc.GetWeaponData then
        local orig_getweapondata = pdc.GetWeaponData
        pdc.GetWeaponData = function(p1, ...)
            local weapon_name = ({...})[1]
            local original_data, index = orig_getweapondata(p1, ...)

            local fake_owned = false
            for _, weapon in ipairs(states.skinchanger_state.fake_weapon_owned) do
                if weapon.Name == weapon_name then fake_owned = true break end
            end

            if fake_owned and not original_data then
                local fake_data = {
                    Name = weapon_name,
                    Level = 1,
                    XP = 0,
                    IsFavorited = false,
                    Skin = nil,
                }
                local equipped = states.skinchanger_state.equipped
                if equipped and equipped[weapon_name] then
                    for cos_type, cos_data in pairs(equipped[weapon_name]) do
                        if type(cos_data) == "table" and (cos_data.Name == "NONE_COSMETIC" or cos_data.Name == "None") then
                            fake_data[cos_type] = nil
                        else
                            fake_data[cos_type] = cos_data
                        end
                    end
                end
                return fake_data, index
            end

            local equipped = states.skinchanger_state.equipped
            if original_data and equipped and equipped[weapon_name] then
                for cos_type, cos_data in pairs(equipped[weapon_name]) do
                    if type(cos_data) == "table" and (cos_data.Name == "NONE_COSMETIC" or cos_data.Name == "None") then
                        original_data[cos_type] = nil
                    else
                        original_data[cos_type] = cos_data
                    end
                end
            end

            return original_data, index
        end
    end
end)

local function PatchCurrentDataGet(current_data)
    if not current_data or type(current_data.Get) ~= "function" or current_data._fakeWeaponOwnedPatched then
        return
    end
    local old_get = current_data.Get
    current_data.Get = function(self, ...)
        local data = old_get(self, ...)
        local key = ({...})[1]
        if key == "CosmeticInventory" then return states.skinchanger_state.fake_owned or data end
        if key == "UnclaimedRewards" then return states.inventory_state.fake_owned or data end
        if key == "WeaponInventory" or key == "FreeWeaponUnlockCheck" then return states.skinchanger_state.fake_weapon_owned or data end
        return data
    end
    current_data._fakeWeaponOwnedPatched = true
end

task.spawn(function()
    do local _t=0 while _t<50 and not (modules.PlayerDataController and modules.PlayerDataController.CurrentData) do _t+=1 task.wait(0.1) end end
    PatchCurrentDataGet(modules.PlayerDataController.CurrentData)
end)

local device_cancel = 0
local function SetDevice(use_old)
    -- AURORA device spoof (oficial Rivals SetControls)
    local map = {
        Desktop = "MouseKeyboard",
        Mobile = "Touch",
        Console = "Gamepad",
        VR = "VR",
        PC = "MouseKeyboard",
        Controller = "Gamepad",
    }
    local key = use_old and states.spoofer_state.old_device or states.spoofer_state.device
    if key == "PC" then key = "Desktop" end
    if key == "Controller" then key = "Console" end
    local payload = map[key] or "MouseKeyboard"
    pcall(function()
        local rem = ReplicatedStorage:FindFirstChild("Remotes")
        local fig = rem and rem:FindFirstChild("Replication") and rem.Replication:FindFirstChild("Fighter")
        local sc = fig and fig:FindFirstChild("SetControls")
        if sc then
            sc:FireServer(payload)
        else
            ReplicatedStorage.Remotes.Replication.Fighter.SetControls:FireServer(payload)
        end
    end)
    pcall(function()
        if modules.ControlsController then
            modules.ControlsController.CurrentControls = key
        end
    end)
    pcall(function()
        LocalPlayer:SetAttribute("Controls", payload)
        LocalPlayer:SetAttribute("Device", tostring(key))
    end)
end

local function SpoofDevice()
    SetDevice(not states.spoofer_state.spoof_device)
end

local function DeviceSpam()
    device_cancel += 1
    local token = device_cancel
    task.spawn(function()
        local list = {"Desktop","Mobile","Console","VR"}
        while states.spoofer_state.device_spam and token == device_cancel do
            states.spoofer_state.device = list[math.random(1, #list)]
            SetDevice(false)
            task.wait(states.spoofer_state.spam_rate)
        end
        SetDevice(true)
    end)
end

local function IsTextObject(obj)
    return obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox")
end

local function FormatNumber(n)
    n = tostring(n)
    return n:reverse():gsub("%d%d%d", "%1,"):reverse():gsub("^,", "")
end

-- anti sssss: plain replace + no reentry
local _nameSpoofBusy = false
local function plainReplaceOnce(str, findStr, repl)
    if not findStr or findStr == "" or not str then return str end
    if findStr == repl then return str end
    local i, j = string.find(str, findStr, 1, true)
    if not i then return str end
    return string.sub(str, 1, i - 1) .. tostring(repl) .. string.sub(str, j + 1)
end

local function ApplyNameSpoof(obj)
    if _nameSpoofBusy then return end
    if not IsTextObject(obj) then return end
    local text = obj.Text
    if typeof(text) ~= "string" or text == "" then return end

    local s = states.spoofer_state
    local realDisp = LocalPlayer.DisplayName
    local realUser = LocalPlayer.Name
    local fakeDisp = tostring(s.display_name_value or "")
    local fakeUser = tostring(s.username_value or "")
    local parentName = obj.Parent and obj.Parent.Name or ""
    local objName = obj.Name or ""

    -- NUNCA tocar stats / racha / wins / level / kills (evita shako.win en racha)
    local blocked = {
        Wins = true, Value = true, Streak = true, Level = true, Kills = true,
        Score = true, ELO = true, Elo = true, Rank = true, Stat = true,
        Amount = true, Count = true, Number = true, XP = true, Glory = true,
        Tokens = true, Keys = true,
    }
    local blockedParent = {
        Streak = true, Level = true, Wins = true, Kills = true, ELO = true,
        Stats = true, Statistics = true, Leaderboard = true,
    }
    if blocked[objName] or blockedParent[parentName] then
        -- Stats SOLO en TU UI (no perfiles / rachas de otros jugadores)
        local function isMyStatUI(o)
            -- LIGERO: solo nombres en la jerarquia (sin GetDescendants = sin lag)
            if not getgenv()._ShakoOtherNames or tick() - (getgenv()._ShakoOtherNamesAt or 0) > 2 then
                local t = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer then
                        t[plr.Name] = true
                        if plr.DisplayName and plr.DisplayName ~= "" then t[plr.DisplayName] = true end
                    end
                end
                getgenv()._ShakoOtherNames = t
                getgenv()._ShakoOtherNamesAt = tick()
            end
            local others = getgenv()._ShakoOtherNames or {}
            local realUser = LocalPlayer.Name
            local realDisp = LocalPlayer.DisplayName or ""
            local uid = tostring(LocalPlayer.UserId)
            local foundMe, foundOther = false, false
            local chain = ""
            local p = o
            for _ = 1, 8 do
                if not p then break end
                local n = p.Name or ""
                local nl = string.lower(n)
                chain = chain .. " " .. nl
                if others[n] then foundOther = true end
                if n == realUser or n == realDisp or n == uid then foundMe = true end
                if p == LocalPlayer or n == "PlayerGui" then break end
                p = p.Parent
            end
            if foundOther then return false end
            -- killfeed / cards de eliminacion: nunca
            if chain:find("killfeed", 1, true) or chain:find("elimination", 1, true)
                or chain:find("killcard", 1, true) or chain:find("deathcard", 1, true)
                or chain:find("feedentry", 1, true) or chain:find("killercard", 1, true) then
                return false
            end
            -- solo tu perfil / stats
            if foundMe then return true end
            if chain:find("profile", 1, true) or chain:find("career", 1, true)
                or chain:find("self", 1, true) or chain:find("local", 1, true) then
                return true
            end
            return false
        end

        if not isMyStatUI(obj) then
            return -- no tocar racha/wins/level de OTRAS personas
        end

        local newText = text
        local lb = s.leaderboard
        local chain = string.lower(objName .. " " .. parentName)
        pcall(function()
            local p = obj.Parent
            for _ = 1, 6 do
                if not p then break end
                chain = chain .. " " .. string.lower(p.Name or "")
                p = p.Parent
            end
        end)
        local isNum = text:match("^%s*[%d%.,]+%s*$") ~= nil
        if lb then
            if lb.Wins and (objName == "Wins" or chain:find("wins", 1, true)) and (isNum or objName == "Wins") then
                newText = FormatNumber(lb.wins_value)
            elseif lb.Level and (parentName == "Level" or objName == "Level" or objName == "Value" or chain:find("level", 1, true))
                and (isNum or parentName == "Level" or objName == "Value")
                and not chain:find("winrate", 1, true) and not chain:find("weapon", 1, true) then
                if isNum or parentName == "Level" or objName == "Value" then
                    newText = FormatNumber(lb.level_value)
                end
            elseif lb.Streak and (parentName == "Streak" or objName == "Streak" or chain:find("streak", 1, true))
                and (isNum or objName == "Value") then
                newText = FormatNumber(lb.streak_value)
            elseif lb.Kills and (parentName == "Kills" or objName == "Kills" or chain:find("kill", 1, true))
                and (isNum or objName == "Value") then
                newText = FormatNumber(lb.kills_value)
            end
        end
        if newText ~= text then
            local num = tonumber(newText) or (typeof(newText) == "string" and tonumber((tostring(newText):gsub("[^%d%-%.]", ""))))
            if num or (typeof(newText) == "string" and not string.find(tostring(newText), "%a")) then
                _nameSpoofBusy = true
                pcall(function() obj.Text = tostring(newText) end)
                _nameSpoofBusy = false
            end
        end
        return
    end

    -- ya es fake completo → no tocar
    if s.display_name and fakeDisp ~= "" and text == fakeDisp then return end
    if s.username and fakeUser ~= "" and text == fakeUser then return end
    if fakeDisp ~= "" and text == fakeDisp .. "\n@" .. (fakeUser ~= "" and fakeUser or realUser) then return end

    local newText = text

    -- SOLO labels de nombre (no Text/Label genericos sueltos)
    local isNameLabel = objName == "DisplayName" or objName == "HeaderText" or objName == "Title"
        or objName == "PlayerName" or objName == "NameLabel"
        or (parentName == "NameContainer" and objName == "Name")
        or (parentName == "Title" and objName == "TitleText")

    local isUserLabel = objName == "Username" or objName == "Handle"
        or (parentName == "NameContainer" and objName == "Handle")
        or (parentName == "Subtitle" and objName == "SubtitleText")

    -- Display name
    if s.display_name and fakeDisp ~= "" and fakeDisp ~= realDisp and realDisp ~= "" and #realDisp >= 2 then
        if isNameLabel and string.find(newText, realDisp, 1, true) and not string.find(newText, fakeDisp, 1, true) then
            newText = plainReplaceOnce(newText, realDisp, fakeDisp)
        elseif (newText == realDisp or newText == realDisp .. "\n@" .. realUser) then
            -- exact match killfeed / ESC
            newText = plainReplaceOnce(newText, realDisp, fakeDisp)
        end
    end

    -- Username
    if s.username and fakeUser ~= "" and fakeUser ~= realUser and realUser ~= "" and #realUser >= 3 then
        if isUserLabel then
            if string.find(newText, "@" .. realUser, 1, true) and not string.find(newText, "@" .. fakeUser, 1, true) then
                newText = plainReplaceOnce(newText, "@" .. realUser, "@" .. fakeUser)
            elseif newText == realUser or newText == "@" .. realUser then
                newText = plainReplaceOnce(newText, realUser, fakeUser)
            end
        elseif newText == "@" .. realUser then
            newText = "@" .. fakeUser
        elseif newText == realUser then
            newText = fakeUser
        end
    end

    if obj.Text ~= newText then
        _nameSpoofBusy = true
        pcall(function() obj.Text = newText end)
        _nameSpoofBusy = false
    end
end

local function RefreshAllNameSpoofs()
    if tick() - (getgenv()._ShakoRefreshNamesAt or 0) < 0.5 then return end
    getgenv()._ShakoRefreshNamesAt = tick()
    local n = 0
    for obj in pairs(states.spoofer_state.name_spoof_conn) do
        if n > 80 then break end
        if obj and obj.Parent then
            pcall(ApplyNameSpoof, obj)
            n = n + 1
        end
    end
end

local function RegisterTextObject(obj)
    if not IsTextObject(obj) then return end
    local allowed = {
        DisplayName=true, Username=true, Handle=true,
        TitleText=true, SubtitleText=true, HeaderText=true,
        PlayerName=true, NameLabel=true,
        -- NO: Name/Text/Label/Value/Wins → contaminan racha y labels genéricos
    }
    if not allowed[obj.Name] then return end
    ApplyNameSpoof(obj)
    if states.spoofer_state.name_spoof_conn[obj] then return end
    -- cap conexiones para no laggear
    local cnt = 0
    for _ in pairs(states.spoofer_state.name_spoof_conn) do
        cnt = cnt + 1
        if cnt > 120 then
            -- limpiar muertos
            for o, c in pairs(states.spoofer_state.name_spoof_conn) do
                if not o or not o.Parent then
                    pcall(function() if c then c:Disconnect() end end)
                    states.spoofer_state.name_spoof_conn[o] = nil
                end
            end
            break
        end
    end
    states.spoofer_state.name_spoof_conn[obj] = obj:GetPropertyChangedSignal("Text"):Connect(function()
        ApplyNameSpoof(obj)
    end)
    obj.Destroying:Connect(function()
        if states.spoofer_state.name_spoof_conn[obj] then
            states.spoofer_state.name_spoof_conn[obj]:Disconnect()
            states.spoofer_state.name_spoof_conn[obj] = nil
        end
    end)
end

-- NO escanear todo el juego (lag). Solo PlayerGui + CoreGui liviano
task.spawn(function()
    local function scanRoot(root, fn)
        if not root then return end
        local n = 0
        for _, o in ipairs(root:GetDescendants()) do
            pcall(fn, o)
            n = n + 1
            if n % 200 == 0 then task.wait() end -- yield anti-freeze
        end
    end
    local pg = LocalPlayer:FindFirstChild("PlayerGui") or LocalPlayer:WaitForChild("PlayerGui", 8)
    scanRoot(pg, RegisterTextObject)
    if pg and not getgenv()._ShakoNamePgHook then
        getgenv()._ShakoNamePgHook = true
        pg.DescendantAdded:Connect(function(o)
            task.defer(RegisterTextObject, o)
        end)
    end
    -- CoreGui solo PlayerList (si existe)
    pcall(function()
        local cg = game:GetService("CoreGui")
        local pl = cg:FindFirstChild("PlayerList")
        if pl then scanRoot(pl, RegisterTextObject) end
    end)
end)

local function ApplyThumbnailSpoof(obj)
    if not obj:IsA("ImageLabel") then return end
    local image = obj.Image
    if typeof(image) ~= "string" then return end

    if states.spoofer_state.spoof_avatar and obj.Name == "AvatarThumbnail" then
        image = image:gsub(LocalPlayer.UserId, states.spoofer_state.avatar_userid)
    end
    if states.spoofer_state.spoof_avatar and obj.Parent.Name == "Thumbnail" and obj.Name == "ImageLabel" then
        image = image:gsub(LocalPlayer.UserId, states.spoofer_state.avatar_userid)
    end
    if states.spoofer_state.spoof_avatar and obj.Name == "Headshot" then
        image = image:gsub(LocalPlayer.UserId, states.spoofer_state.avatar_userid)
    end

    if obj.Image ~= image then obj.Image = image end
end

local function RefreshAllThumbnailSpoofs()
    for obj in pairs(states.spoofer_state.thumb_spoof_conn) do
        if obj and obj.Parent then ApplyThumbnailSpoof(obj) end
    end
end

local function RegisterImageObject(obj)
    if not obj:IsA("ImageLabel") then return end
    if not ({AvatarThumbnail=true, ImageLabel=true, Headshot=true})[obj.Name] then return end
    ApplyThumbnailSpoof(obj)
    if states.spoofer_state.thumb_spoof_conn[obj] then return end
    states.spoofer_state.thumb_spoof_conn[obj] = obj:GetPropertyChangedSignal("Image"):Connect(function()
        ApplyThumbnailSpoof(obj)
    end)
    obj.Destroying:Connect(function()
        if states.spoofer_state.thumb_spoof_conn[obj] then
            states.spoofer_state.thumb_spoof_conn[obj]:Disconnect()
            states.spoofer_state.thumb_spoof_conn[obj] = nil
        end
    end)
end

task.spawn(function()
    local function scanRoot(root, fn)
        if not root then return end
        local n = 0
        for _, o in ipairs(root:GetDescendants()) do
            pcall(fn, o)
            n = n + 1
            if n % 200 == 0 then task.wait() end
        end
    end
    local pg = LocalPlayer:FindFirstChild("PlayerGui") or LocalPlayer:WaitForChild("PlayerGui", 8)
    scanRoot(pg, RegisterImageObject)
    if pg and not getgenv()._ShakoImgPgHook then
        getgenv()._ShakoImgPgHook = true
        pg.DescendantAdded:Connect(function(o)
            task.defer(RegisterImageObject, o)
        end)
    end
end)

local function FindFirstMatchingAttachment(model, name)
    for _, child in pairs(model:GetChildren()) do
        if child:IsA("Attachment") and child.Name == name then return child end
        if not child:IsA("Accoutrement") and not child:IsA("Tool") then
            local f = FindFirstMatchingAttachment(child, name)
            if f then return f end
        end
    end
end

local function BuildWeld(weld_name, parent, part0, part1, c0, c1)
    local weld = Instance.new("Weld")
    weld.Name = weld_name
    weld.Part0 = part0
    weld.Part1 = part1
    weld.C0 = c0
    weld.C1 = c1
    weld.Parent = parent
    return weld
end

local function WeldAttachments(a1, a2)
    local w = Instance.new("Weld")
    w.Part0, w.Part1 = a1.Parent, a2.Parent
    w.C0, w.C1 = a1.CFrame, a2.CFrame
    w.Parent = a1.Parent
    return w
end

local function AddAccoutrement(character, acc)
    acc.Parent = character
    local handle = acc:FindFirstChild("Handle")
    if not handle then return end
    local att = handle:FindFirstChildOfClass("Attachment")
    if att then
        local ca = FindFirstMatchingAttachment(character, att.Name)
        if ca then WeldAttachments(ca, att) end
    else
        local head = character:FindFirstChild("Head")
        if head then
            local attachmentCFrame = CFrame.new(0, 0.5, 0)
            local hatCFrame = acc.AttachmentPoint
            BuildWeld("HeadWeld", head, head, handle, attachmentCFrame, hatCFrame)
        end
    end
end

local function FindHeadPart(inst)
    if not inst then return end
    if inst:IsA("BasePart") then return inst end
    local h = inst:FindFirstChild("Head")
    if h and h:IsA("BasePart") then return h end
    for _, d in ipairs(inst:GetDescendants()) do
        if d:IsA("BasePart") and d.Name == "Head" then return d end
    end
    for _, d in ipairs(inst:GetDescendants()) do
        if d:IsA("BasePart") then return d end
    end
end

local function CopyHeadVisuals(real_head, source_head)
    if not real_head or not source_head then return end
    if not real_head:IsA("BasePart") or not source_head:IsA("BasePart") then return end
    for _, c in ipairs(real_head:GetChildren()) do
        if c:IsA("Decal") or c:IsA("Texture") or c:IsA("SpecialMesh") or c:IsA("Mesh") then c:Destroy() end
    end
    real_head.Material = source_head.Material
    real_head.Color = source_head.Color
    real_head.Reflectance = source_head.Reflectance
    real_head.Transparency = source_head.Transparency
    real_head.LocalTransparencyModifier = source_head.LocalTransparencyModifier
    real_head.CastShadow = source_head.CastShadow
    real_head.CanCollide = false
    if real_head:IsA("MeshPart") and source_head:IsA("MeshPart") then
        real_head.MeshId = source_head.MeshId
        real_head.TextureID = source_head.TextureID
        real_head.Size = source_head.Size
    end
    for _, c in ipairs(source_head:GetChildren()) do
        if c:IsA("Decal") or c:IsA("Texture") or c:IsA("SpecialMesh") or c:IsA("Mesh") then
            c:Clone().Parent = real_head
        end
    end
end

local function ApplyFakeHead(character, source_head)
    local real = character and character:FindFirstChild("Head")
    if not real or not real:IsA("BasePart") or not source_head then return end
    CopyHeadVisuals(real, source_head)
end

local function SetAvatar(use_old)
    -- Vnix.lol style: HumanoidDescription primero (ropa + accesorios estables)
    local uid = use_old and LocalPlayer.UserId or states.spoofer_state.avatar_userid
    if not uid then return end
    uid = tonumber(uid) or uid
    if not LocalChar or not LocalChar.Parent then LocalChar = GetCharacter() end
    if not LocalChar then return end
    local hum = LocalChar:FindFirstChildOfClass("Humanoid")
    if not hum then return end

    local okDesc, desc = pcall(function()
        return Players:GetHumanoidDescriptionFromUserId(uid)
    end)
    if okDesc and desc then
        pcall(function()
            if hum.ApplyDescriptionReset then
                hum:ApplyDescriptionReset(desc)
            else
                hum:ApplyDescription(desc)
            end
        end)
        return
    end

    -- fallback: clone model (viejo metodo)
    local ok, rig = pcall(function() return Players:CreateHumanoidModelFromUserId(uid) end)
    if not ok or not rig then return end
    for _, v in ipairs(LocalChar:GetDescendants()) do
        if v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic")
            or v:IsA("BodyColors") or v:IsA("CharacterMesh") or v:IsA("LayeredClothing")
            or (v:IsA("Decal") and v.Name == "face") then
            pcall(function() v:Destroy() end)
        end
    end
    for _, item in ipairs(rig:GetChildren()) do
        if item:IsA("Accessory") then
            pcall(function() AddAccoutrement(LocalChar, item:Clone()) end)
        elseif item:IsA("Shirt") or item:IsA("Pants") or item:IsA("ShirtGraphic")
            or item:IsA("BodyColors") or item:IsA("LayeredClothing") or item:IsA("CharacterMesh") then
            pcall(function() item:Clone().Parent = LocalChar end)
        end
    end
    local head = FindHeadPart(rig)
    if head then ApplyFakeHead(LocalChar, head) end
    pcall(function() rig:Destroy() end)
end

local function SpoofAvatar()
    SetAvatar(not states.spoofer_state.spoof_avatar)
end

-- Reaplicar avatar spoof tras respawn / match / campo de tiro (Rivals limpia ropa)
local function ReapplyAvatarSpoofDelayed()
    if not states or not states.spoofer_state or not states.spoofer_state.spoof_avatar then return end
    local uid = tonumber(states.spoofer_state.avatar_userid) or tonumber(config.SkinTargetUserId)
    if not uid or uid <= 0 then return end
    task.spawn(function()
        -- varios intentos: Rivals tira la skin a los ~0.5–3s al entrar al range/partida
        for _, delay in ipairs({0.4, 1.0, 2.0, 3.5, 5.5}) do
            task.wait(delay)
            if not states.spoofer_state.spoof_avatar then return end
            pcall(function() SetAvatar(false) end)
            if Skin and Skin.copyAvatar then
                pcall(function() Skin.copyAvatar(uid) end)
            end
        end
    end)
end

if not getgenv()._ShakoAvatarCharHook then
    getgenv()._ShakoAvatarCharHook = true
    LocalPlayer.CharacterAdded:Connect(function(char)
        ReapplyAvatarSpoofDelayed()
        -- si el juego borra ropa/accesorios, volver a ponerlos
        task.defer(function()
            task.wait(0.2)
            if not char or not char.Parent then return end
            if getgenv()._ShakoAvatarChildHook then return end
            -- un solo hook por sesión sobre el character actual
            local conn
            conn = char.ChildRemoved:Connect(function(child)
                if not states.spoofer_state.spoof_avatar then return end
                if child:IsA("Accessory") or child:IsA("Hat") or child:IsA("Shirt")
                    or child:IsA("Pants") or child:IsA("ShirtGraphic") or child:IsA("BodyColors") then
                    if getgenv()._ShakoAvatarReapplyBusy then return end
                    getgenv()._ShakoAvatarReapplyBusy = true
                    task.delay(0.35, function()
                        getgenv()._ShakoAvatarReapplyBusy = false
                        if states.spoofer_state.spoof_avatar then
                            pcall(function() SetAvatar(false) end)
                            local uid = tonumber(states.spoofer_state.avatar_userid) or tonumber(config.SkinTargetUserId)
                            if uid and Skin and Skin.copyAvatar then
                                pcall(function() Skin.copyAvatar(uid) end)
                            end
                        end
                    end)
                end
            end)
            char.AncestryChanged:Connect(function(_, parent)
                if not parent and conn then pcall(function() conn:Disconnect() end) end
            end)
        end)
    end)
end

local function GenerateRandomName()
    local result = ""
    for i = 1, math.random(2, 4) do
        local index = math.random(1, #kWordList)
        result ..= kWordList[index]
    end
    return result .. tostring(math.random(12, 36))
end

local function GetFakeName(player)
    if not states.spoofer_state.fake_names[player] then
        states.spoofer_state.fake_names[player] = GenerateRandomName()
    end
    return states.spoofer_state.fake_names[player]
end

local AnonymousRigTemplate
local function GetAnonymousRig()
    if AnonymousRigTemplate and AnonymousRigTemplate.Parent then
        return AnonymousRigTemplate:Clone()
    end
    local ok, rig = pcall(function()
        return Players:CreateHumanoidModelFromUserId(6942067)
    end)
    if not ok or not rig then return nil end
    AnonymousRigTemplate = rig
    AnonymousRigTemplate.Parent = nil
    return AnonymousRigTemplate:Clone()
end

local function ApplyAnonymousNameSpoof(obj)
    if not IsTextObject(obj) then return end
    local text = obj.Text
    for _, player in ipairs(Players:GetPlayers()) do
        local fakeName = GetFakeName(player)
        text = text:gsub(player.Name, fakeName)
        text = text:gsub(player.DisplayName, fakeName)
    end
    if obj.Text ~= text then obj.Text = text end
end

local function ApplyAnonymousThumbnailSpoof(obj)
    if not obj:IsA("ImageLabel") then return end
    local image = obj.Image
    for _, player in ipairs(Players:GetPlayers()) do
        image = image:gsub(tostring(player.UserId), tostring(6942067))
    end
    if obj.Image ~= image then obj.Image = image end
end

local function ApplyAnonymousAvatar(character)
    if not character then return end
    local rig = GetAnonymousRig()
    if not rig then return end
    for _, v in ipairs(character:GetDescendants()) do
        local isFaceDecal = v:IsA("Decal") and v.Name == "face"
        if v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic")
            or v:IsA("BodyColors") or v:IsA("CharacterMesh") or v:IsA("LayeredClothing")
            or isFaceDecal then
            pcall(function() v:Destroy() end)
        end
    end
    for _, item in ipairs(rig:GetChildren()) do
        if item:IsA("Accessory") then
            AddAccoutrement(character, item:Clone())
        elseif item:IsA("Shirt") or item:IsA("Pants") or item:IsA("ShirtGraphic")
            or item:IsA("BodyColors") or item:IsA("CharacterMesh") or item:IsA("LayeredClothing") then
            item:Clone().Parent = character
        end
    end
    local head = FindHeadPart(rig)
    if head then ApplyFakeHead(character, head) end
    rig:Destroy()
end

local function RegisterAnonymousObject(obj)
    if IsTextObject(obj) then
        ApplyAnonymousNameSpoof(obj)
        obj:GetPropertyChangedSignal("Text"):Connect(function()
            ApplyAnonymousNameSpoof(obj)
        end)
    elseif obj:IsA("ImageLabel") then
        ApplyAnonymousThumbnailSpoof(obj)
        obj:GetPropertyChangedSignal("Image"):Connect(function()
            ApplyAnonymousThumbnailSpoof(obj)
        end)
    end
end

local function DisableAnonymousMode()
    for _, c in ipairs(states.spoofer_state.anon_connections) do
        if c and c.Disconnect then
            pcall(function() c:Disconnect() end)
        end
    end
    table.clear(states.spoofer_state.anon_connections)
    table.clear(states.spoofer_state.fake_names)
end

local function AnonymousMode()
    if not states.spoofer_state.anonymous_mode then
        DisableAnonymousMode()
        return
    end
    DisableAnonymousMode()
    math.randomseed(os.clock() * 100000)
    for _, player in ipairs(Players:GetPlayers()) do
        GetFakeName(player)
        if player.Character then
            task.defer(ApplyAnonymousAvatar, player.Character)
        end
        table.insert(states.spoofer_state.anon_connections,
            player.CharacterAdded:Connect(function(character)
                task.defer(ApplyAnonymousAvatar, character)
            end))
    end
    table.insert(states.spoofer_state.anon_connections,
        Players.PlayerAdded:Connect(function(player)
            GetFakeName(player)
            player.CharacterAdded:Connect(function(character)
                task.defer(ApplyAnonymousAvatar, character)
            end)
        end))
    task.spawn(function()
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg then
            local n = 0
            for _, obj in ipairs(pg:GetDescendants()) do
                pcall(RegisterAnonymousObject, obj)
                n = n + 1
                if n % 150 == 0 then task.wait() end
            end
            table.insert(states.spoofer_state.anon_connections,
                pg.DescendantAdded:Connect(function(o) task.defer(RegisterAnonymousObject, o) end))
        end
    end)
end

local function UpdateBadgesOptions()
    LocalPlayer:SetAttribute("IsInfluencer", states.spoofer_state.badges.Influencer and true or false)
    LocalPlayer:SetAttribute("IsRobloxEmployee", states.spoofer_state.badges.Admin and true or false)
    -- Premium/Verified are usually membership flags; attribute spoof covers common UI checks
    LocalPlayer:SetAttribute("IsPremium", states.spoofer_state.badges.Premium and true or false)
    LocalPlayer:SetAttribute("HasVerifiedBadge", states.spoofer_state.badges.Verified and true or false)
end

pcall(function()
    local old_badge_index
    old_badge_index = hookmetamethod(game, "__index", function(self, key)
        if self == LocalPlayer then
            if key == "MembershipType" and states.spoofer_state.badges.Premium then
                return Enum.MembershipType.Premium
            end
            if key == "HasVerifiedBadge" and states.spoofer_state.badges.Verified then
                return true
            end
        end
        return old_badge_index(self, key)
    end)
end)

local function InsertPlayerByValue(list, user_id, value)
    for i = #list, 1, -1 do
        if tonumber(list[i].key) == user_id then table.remove(list, i) break end
    end
    local rank = 1
    for i, e in ipairs(list) do
        if tonumber(e.value) >= value then rank = i + 1 else break end
    end
    table.insert(list, rank, {key = tostring(user_id), value = value})
end

local function ResetLeaderboard(name, config_idx)
    if not modules.LeaderboardController then return end
    Unbind(config_idx)
    local serial = modules.LeaderboardController.LeaderboardSerials
        and modules.LeaderboardController.LeaderboardSerials[name]
    if serial and serial.Players then
        local uid = LocalPlayer.UserId
        for i = #serial.Players, 1, -1 do
            local p = serial.Players[i]
            if p.UserId == uid or p.UserID == uid or p.PlayerId == uid then
                table.remove(serial.Players, i)
            end
        end
        if modules.LeaderboardController.Refreshed then
            modules.LeaderboardController.Refreshed:Fire(name)
        end
    end
end

local function ModifyLeaderboard(config_idx, name, value)
    if not modules.LeaderboardController then return end
    task.spawn(function()
        while not modules.LeaderboardController.LeaderboardSerials[name]
            or not modules.LeaderboardController.LeaderboardSerials[name].Players
            or #modules.LeaderboardController.LeaderboardSerials[name].Players <= 0 do
            task.wait(0.5)
        end

        local user_id = LocalPlayer.UserId
        local serial = modules.LeaderboardController.LeaderboardSerials[name]
        if serial and serial.Players then
            InsertPlayerByValue(serial.Players, user_id, value)
            if modules.LeaderboardController.Refreshed then
                modules.LeaderboardController.Refreshed:Fire(name)
            end
        end

        local UpdateRemote = ReplicatedStorage:FindFirstChild("Remotes")
            and ReplicatedStorage.Remotes:FindFirstChild("Misc")
            and ReplicatedStorage.Remotes.Misc:FindFirstChild("UpdateLeaderboard")
        if not (UpdateRemote and UpdateRemote:IsA("RemoteEvent")) then return end

        Bind(config_idx, UpdateRemote.OnClientEvent, function(data)
            local lb_name = data and data.Name
            if not lb_name or lb_name ~= name then return end
            if not states.spoofer_state.leaderboard[config_idx] then return end
            task.defer(function()
                local serial = modules.LeaderboardController.LeaderboardSerials[name]
                if serial and serial.Players then
                    InsertPlayerByValue(serial.Players, user_id, value)
                    if modules.LeaderboardController.Refreshed then
                        modules.LeaderboardController.Refreshed:Fire(name)
                    end
                end
            end)
        end)
    end)
end

local function UpdateLeaderboardOptions()
    local lb = states.spoofer_state.leaderboard

    if lb.ELO then
        Unbind("ELO")
        ModifyLeaderboard("ELO", "Highest ELO", lb.elo_value)
    end

    if lb.Streak then
        Unbind("Streak")
        ModifyLeaderboard("Streak", "Current Highest Win Streak", lb.streak_value)
        lb.old_streak_value = LocalPlayer:GetAttribute("StatisticDuelsWinStreak")
        LocalPlayer:SetAttribute("StatisticDuelsWinStreak", lb.streak_value)
    end

    if lb.Kills then
        Unbind("Kills")
        ModifyLeaderboard("Kills", "Most Eliminations", lb.kills_value)
    end

    if lb.Wins then
        Unbind("Wins")
        ModifyLeaderboard("Wins", "Most Wins", lb.wins_value)
    end

    if lb.Level then
        Unbind("Level")
        ModifyLeaderboard("Level", "Highest Level", lb.level_value)
        lb.old_level_value = LocalPlayer:GetAttribute("Level")
        LocalPlayer:SetAttribute("Level", lb.level_value)
    end

    if not lb.ELO then ResetLeaderboard("Highest ELO", "ELO") end

    if not lb.Streak then
        ResetLeaderboard("Current Highest Win Streak", "Streak")
        if lb.old_streak_value ~= nil then
            LocalPlayer:SetAttribute("StatisticDuelsWinStreak", lb.old_streak_value)
        end
    end

    if not lb.Kills then ResetLeaderboard("Most Eliminations", "Kills") end

    if not lb.Wins then ResetLeaderboard("Most Wins", "Wins") end

    if not lb.Level then
        ResetLeaderboard("Highest Level", "Level")
        if lb.old_level_value ~= nil then
            LocalPlayer:SetAttribute("Level", lb.old_level_value)
        end
    end
end

-- Rank/Charm hook (método AURORA exacto)
local _charmHooked = false
local function HookSeasonRankCharm()
    if not modules.SeasonLibrary then
        pcall(function()
            local RSModules = ReplicatedStorage:FindFirstChild("Modules")
            if RSModules then
                modules.SeasonLibrary = require(RSModules:WaitForChild("SeasonLibrary", 3))
            end
        end)
    end
    if not modules.SeasonLibrary or not modules.SeasonLibrary.FormatSeasonRankCharm then return false end
    if _charmHooked then return true end
    local original = modules.SeasonLibrary.FormatSeasonRankCharm
    modules.SeasonLibrary.FormatSeasonRankCharm = function(self, p2, p3, p4, p5)
        local s = states.spoofer_state
        local season_num = kSeasonNameMap[p3]
        local rankName = s.charm and s.charm.charm_rank
        local elo = kCharmEloMap[rankName]
        if not elo and rankName ~= "Use Spoofed ELO" then
            elo = nil
        end
        elo = elo or (s.leaderboard and s.leaderboard.elo_value) or p4
        -- si hay rank spoof activo, aplicar a TODAS las seasons (o solo las toggled)
        local anySeason = s.charm and (s.charm.s0_charm or s.charm.s1_charm or s.charm.s2_charm or s.charm.s3_charm)
        local thisSeason = season_num and kSeasonToggleMap[season_num] and s.charm and s.charm[kSeasonToggleMap[season_num]]
        if rankName and rankName ~= "Use Spoofed ELO" and (thisSeason or anySeason) then
            p4 = elo or p4
            p5 = (s.charm and s.charm.arch_rank) or p5
        end
        return original(self, p2, p3, p4, p5)
    end
    _charmHooked = true
    return true
end
pcall(function() if getgenv()._ShakoHookCharm then getgenv()._ShakoHookCharm() elseif HookSeasonRankCharm then HookSeasonRankCharm() end end)
getgenv()._ShakoHookCharm = HookSeasonRankCharm

local function UpdateCharmOptions()
    task.spawn(function()
        -- re-hook por si módulos cargaron tarde
        pcall(HookSeasonRankCharm)
        -- sync RivalsSpoof → states
        pcall(function()
            if RivalsSpoof and RivalsSpoof.charm and states.spoofer_state then
                local c = RivalsSpoof.charm
                local sc = states.spoofer_state.charm
                if c.charm_rank then sc.charm_rank = c.charm_rank end
                if c.arch_rank then sc.arch_rank = c.arch_rank end
                for _, k in ipairs({"s0_charm","s1_charm","s2_charm","s3_charm"}) do
                    if c[k] ~= nil then sc[k] = c[k] and true or false end
                end
            end
        end)
        local s = states.spoofer_state
        if not s or not s.charm then return end
        local rankName = s.charm.charm_rank or "Archnemesis"
        local elo = kCharmEloMap[rankName] or s.leaderboard.elo_value or 3600
        local arch = s.charm.arch_rank or 1

        -- asegurar ELO leaderboard
        if s.leaderboard then
            s.leaderboard.ELO = true
            s.leaderboard.elo_value = elo
        end
        pcall(function()
            if UpdateLeaderboardOptions then UpdateLeaderboardOptions() end
        end)

        local function applyToModel(d, season_num)
            if not d:IsA("Model") then return end
            if not kSeasonToggleMap[season_num] then return end
            if not s.charm[kSeasonToggleMap[season_num]] then return end
            local season_name = kSeasonNameMap[season_num]
            if not season_name or not modules.SeasonLibrary then return end
            pcall(function()
                modules.SeasonLibrary:FormatSeasonRankCharm(d, season_name, elo, arch)
            end)
        end

        local function scan(root)
            for _, d in ipairs(root:GetDescendants()) do
                if d:IsA("Model") then
                    local name = d.Name
                    if string.sub(name, 1, 7) == "Season " then
                        local season_num = tonumber(string.sub(name, 8))
                        if season_num then applyToModel(d, season_num) end
                    end
                end
            end
        end

        -- aplicar ya + retries (modelos de season tardan en spawnear)
        for attempt = 1, 2 do
            pcall(function()
                local pg = LocalPlayer:FindFirstChild("PlayerGui")
                if pg then scan(pg) end
            end)
            -- workspace: solo Models con Extra (charm), con yield
            pcall(function()
                local n = 0
                for _, d in ipairs(workspace:GetChildren()) do
                    if d:IsA("Model") then
                        local nm = d.Name
                        if string.sub(nm, 1, 7) == "Season " then
                            local sn = tonumber(string.sub(nm, 8))
                            if sn then applyToModel(d, sn) end
                        end
                    end
                    n = n + 1
                    if n % 40 == 0 then task.wait() end
                end
            end)
            task.wait(0.5)
        end
    end)
end

local function UpdateCurrencyOptions()
    pcall(function()
        local cd = modules.PlayerDataController and modules.PlayerDataController.CurrentData
        if not cd then return end
        cd:Replicate("WeaponKeys")
        cd:Replicate("UnlockTokens")
        cd:Replicate("EventCurrency")
        cd:Replicate("Glory")
        cd:Replicate("SkinTickets")
    end)
end

local function UpdateDuelHistory()
    local dh = states.spoofer_state.duel_history
    local list = dh.duel_history_value
    if type(list) ~= "table" then return end
    local match = list[dh.match_index]
    if not match then return end

    match.QueueName = dh.mode == "Ranked" and "Ranked" or "Casual"
    match.Map = dh.map
    match.Scores = {dh.team1score, dh.team2score}
    match.WinningTeamIndex = dh.result == "Victory" and 1 or 2

    if match.Duelers and match.Duelers[dh.dueler_index] then
        local me = match.Duelers[dh.dueler_index]
        me.Username = dh.username
        me.DisplayName = dh.display_name
        me.Eliminations = dh.kills
        me.Deaths = dh.deaths
        me.Assists = dh.assists
        me.Damage = dh.damage
        if dh.mode == "Ranked" then
            me.DisplayELO = dh.elo
            if type(dh.logged_elo_events) == "table" and dh.logged_elo_events[dh.match_index] then
                dh.logged_elo_events[dh.match_index].CurrentELO = dh.elo
                dh.logged_elo_events[dh.match_index].ELOIncrement = dh.elo_change or 0
            end
        end
    end

    pcall(function()
        local cd = modules.PlayerDataController and modules.PlayerDataController.CurrentData
        if cd then
            cd:Replicate("DuelHistory")
            cd:Replicate("LoggedELOEvents")
        end
    end)
end

local NamecallDispatcher = {
    Hooks = {},
    Original = nil,
}

function NamecallDispatcher:Register(callback)
    self.Hooks[#self.Hooks + 1] = callback
    return #self.Hooks
end

function NamecallDispatcher:Unregister(index)
    table.remove(self.Hooks, index)
end

function NamecallDispatcher:Clear()
    table.clear(self.Hooks)
end

function NamecallDispatcher:CallOriginal(object, ...)
    if self.Original then return self.Original(object, ...) end
    return nil
end

pcall(function()
    NamecallDispatcher.Original = hookmetamethod(game, "__namecall", newcclosure(function(object, ...)
        local hooks = NamecallDispatcher.Hooks
        local count = #hooks
        if count == 0 then return NamecallDispatcher.Original(object, ...) end
        local method = getnamecallmethod()
        for i = 1, count do
            local result = hooks[i](object, method, ...)
            if result ~= nil and result ~= false then
                if result == true then return nil end
                return result
            end
        end
        return NamecallDispatcher.Original(object, ...)
    end))
end)

local function CloneCosmetic(name, cosmetic_type, options)
    if not modules.CosmeticLibrary or not modules.CosmeticLibrary.Cosmetics then return nil end
    local base = modules.CosmeticLibrary.Cosmetics[name]
    if not base then return nil end
    local data = table.clone(base)
    data.Name = name
    data.Type = data.Type or cosmetic_type
    data.Seed = math.random(1, 1000000)
    if options then
        if options.inverted then data.Inverted = true end
        if options.favorites_only then data.OnlyUseFavorites = true end
    end
    return data
end

local function HandleSkinchangerEquip(args)
    local weapon_name = args[1]
    local cosmetic_type = args[2]
    local cosmetic_name = args[3]
    local options = args[4] or {}

    states.skinchanger_state.equipped[weapon_name] = states.skinchanger_state.equipped[weapon_name] or {}

    if not cosmetic_name or cosmetic_name == "" or cosmetic_name == "None" or cosmetic_name == "NONE_COSMETIC" then
        states.skinchanger_state.equipped[weapon_name][cosmetic_type] = { Name = "NONE_COSMETIC" }
    elseif cosmetic_name == "Random" or cosmetic_name == "RANDOM_COSMETIC" then
        states.skinchanger_state.equipped[weapon_name][cosmetic_type] = {
            Name = cosmetic_name,
            Type = cosmetic_type,
            Inverted = options.IsInverted,
            OnlyUseFavorites = options.OnlyUseFavorites,
            Seed = math.random(1, 1000000),
        }
    else
        local cloned = CloneCosmetic(cosmetic_name, cosmetic_type, {
            inverted = options.IsInverted,
            favorites_only = options.OnlyUseFavorites,
        })
        if cloned then
            states.skinchanger_state.equipped[weapon_name][cosmetic_type] = cloned
        end
    end

    task.spawn(function()
        task.wait(0.1)
        pcall(function()
            if modules.PlayerDataController and modules.PlayerDataController.CurrentData then
                modules.PlayerDataController.CurrentData:Replicate("WeaponInventory")
            end
        end)
    end)
end

local function HandleSkinchangerFavorite(args)
    local weapon_name = args[1]
    local cosmetic_name = args[2]
    local state = args[3]
    states.skinchanger_state.favorites = states.skinchanger_state.favorites or {}
    states.skinchanger_state.favorites[weapon_name] = states.skinchanger_state.favorites[weapon_name] or {}
    states.skinchanger_state.favorites[weapon_name][cosmetic_name] = state or nil
end

pcall(function()
    NamecallDispatcher:Register(function(self, method, ...)
        if method ~= "FireServer" then return false end
        local args = {...}
        if self.Name == "EquipCosmetic" then
            HandleSkinchangerEquip(args)
            return true
        end
        if self.Name == "FavoriteCosmetic" then
            HandleSkinchangerFavorite(args)
            return true
        end
        return false
    end)
end)

local function ResolveCosmetic(weaponName, cosmeticType)
    local equipped = states.skinchanger_state.equipped[weaponName]
        and states.skinchanger_state.equipped[weaponName][cosmeticType]
    if not equipped then return nil end
    if equipped.Name == "None" or equipped.Name == "NONE_COSMETIC" then return nil end
    return equipped
end

pcall(function()
    if not modules.ItemLibrary then return end
    local orig = modules.ItemLibrary.GetViewModelImageFromWeaponData
    modules.ItemLibrary.GetViewModelImageFromWeaponData = function(p1, p2, p3)
        if not p2 then return orig(p1, p2, p3) end
        local weapon_name = p2.Name
        local skin = ResolveCosmetic(weapon_name, "Skin")
        local should_show = skin and (
            (p2.Skin and states.skinchanger_state.equipped[weapon_name])
            or (states.skinchanger_state.viewing_profile == LocalPlayer and states.skinchanger_state.equipped[weapon_name])
        )
        if should_show and skin then
            local skin_info = p1.ViewModels[skin.Name]
            if skin_info then
                return skin_info[p3 and "ImageHighResolution" or "Image"] or skin_info.Image
            end
        end
        return orig(p1, p2, p3)
    end
end)

pcall(function()
    local ok, clientItemPath = pcall(function()
        return LocalPlayer.PlayerScripts.Modules.ClientReplicatedClasses.ClientFighter.ClientItem
    end)
    if not ok or not clientItemPath then return end
    local ClientItem = require(clientItemPath)

    if ClientItem._CreateViewModel then
        local orig = ClientItem._CreateViewModel
        ClientItem._CreateViewModel = function(self, viewmodelRef)
            local weaponName = self.Name
            local weaponPlayer = self.ClientFighter and self.ClientFighter.Player
            states.skinchanger_state.constructing_weapon = (weaponPlayer == LocalPlayer) and weaponName or nil

            if weaponPlayer == LocalPlayer and states.skinchanger_state.equipped[weaponName] and ResolveCosmetic(weaponName, "Skin") and viewmodelRef then
                pcall(function()
                    local ReplicatedClass = require(ReplicatedStorage.Modules.ReplicatedClass)
                    local dataKey = ReplicatedClass:ToEnum("Data")
                    local skinKey = ReplicatedClass:ToEnum("Skin")
                    local nameKey = ReplicatedClass:ToEnum("Name")

                    if viewmodelRef[dataKey] then
                        viewmodelRef[dataKey][skinKey] = ResolveCosmetic(weaponName, "Skin")
                        viewmodelRef[dataKey][nameKey] = ResolveCosmetic(weaponName, "Skin").Name
                    elseif viewmodelRef.Data then
                        viewmodelRef.Data.Skin = ResolveCosmetic(weaponName, "Skin")
                        viewmodelRef.Data.Name = ResolveCosmetic(weaponName, "Skin").Name
                    end
                end)
            end

            local result = orig(self, viewmodelRef)
            states.skinchanger_state.constructing_weapon = nil
            return result
        end
    end

    pcall(function()
        local viewModelModule = LocalPlayer.PlayerScripts.Modules.ClientReplicatedClasses.ClientFighter.ClientItem:FindFirstChild("ClientViewModel")
        if not viewModelModule then return end
        local ClientViewModel = require(viewModelModule)

        if ClientViewModel.GetWrap then
            local origGetWrap = ClientViewModel.GetWrap
            ClientViewModel.GetWrap = function(self)
                local weaponName = self.ClientItem and self.ClientItem.Name
                local weaponPlayer = self.ClientItem and self.ClientItem.ClientFighter and self.ClientItem.ClientFighter.Player
                if weaponName and weaponPlayer == LocalPlayer and states.skinchanger_state.equipped[weaponName] and ResolveCosmetic(weaponName, "Wrap") then
                    return ResolveCosmetic(weaponName, "Wrap")
                end
                return origGetWrap(self)
            end
        end

        local origNew = ClientViewModel.new
        ClientViewModel.new = function(replicatedData, clientItem)
            local weaponPlayer = clientItem.ClientFighter and clientItem.ClientFighter.Player
            local weaponName = states.skinchanger_state.constructing_weapon or clientItem.Name

            if weaponPlayer == LocalPlayer and states.skinchanger_state.equipped[weaponName] then
                pcall(function()
                    local ReplicatedClass = require(ReplicatedStorage.Modules.ReplicatedClass)
                    local dataKey = ReplicatedClass:ToEnum("Data")
                    replicatedData[dataKey] = replicatedData[dataKey] or {}

                    if ResolveCosmetic(weaponName, "Skin") then
                        replicatedData[dataKey][ReplicatedClass:ToEnum("Skin")] = ResolveCosmetic(weaponName, "Skin")
                    end
                    if ResolveCosmetic(weaponName, "Wrap") then
                        replicatedData[dataKey][ReplicatedClass:ToEnum("Wrap")] = ResolveCosmetic(weaponName, "Wrap")
                    end
                    if ResolveCosmetic(weaponName, "Charm") then
                        replicatedData[dataKey][ReplicatedClass:ToEnum("Charm")] = ResolveCosmetic(weaponName, "Charm")
                    end
                end)
            end

            local result = origNew(replicatedData, clientItem)

            pcall(function()
                local objectID = nil
                pcall(function()
                    local ReplicatedClass = require(ReplicatedStorage.Modules.ReplicatedClass)
                    local dataKey = ReplicatedClass:ToEnum("Data")
                    local objKey = ReplicatedClass:ToEnum("ObjectID")
                    if replicatedData[dataKey] then
                        objectID = replicatedData[dataKey][objKey]
                    end
                end)
                if objectID == nil then
                    pcall(function() objectID = result:Get("ObjectID") or result.ObjectID end)
                end
                if objectID and weaponPlayer == LocalPlayer and states.skinchanger_state.equipped[weaponName] then
                    states.skinchanger_state.placed_object_map[objectID] = weaponName
                end
            end)

            if weaponPlayer == LocalPlayer and states.skinchanger_state.equipped[weaponName] and ResolveCosmetic(weaponName, "Wrap") and result._UpdateWrap then
                task.spawn(function()
                    result:_UpdateWrap()
                    task.wait(0.1)
                    if not result._destroyed then result:_UpdateWrap() end
                end)
            end

            return result
        end
    end)
end)

local function FindContainerWithRobuxPrice()
    for _, child in ipairs(CoreGui:GetChildren()) do
        local found = child:FindFirstChild("RobuxPrice", true)
        if found then return child end
    end
    return nil
end

local function AddGamepass(gamepass_id)
    states.spoofer_state.marketplace.fake_bundles_owned[modules.MonetizationLibrary:GetGamepassName(gamepass_id)] = true
    pcall(function()
        if modules.PlayerDataController and modules.PlayerDataController.CurrentData then
            modules.PlayerDataController.CurrentData:Replicate("GamepassBundlesClaimed")
        end
    end)
end

local function AddReward(reward_name, reward_amount, reward_weapon)
    if reward_name == "Key" then
        states.spoofer_state.currency.weapon_keys_value += reward_amount
        pcall(function()
            if modules.PlayerDataController and modules.PlayerDataController.CurrentData then
                modules.PlayerDataController.CurrentData:Replicate("WeaponKeys")
            end
        end)
    elseif modules.CosmeticLibrary and modules.CosmeticLibrary.Cosmetics and modules.CosmeticLibrary.Cosmetics[reward_name] then
        local reward_info = modules.CosmeticLibrary.Cosmetics[reward_name]
        if reward_info.Type == "Skin" or reward_info.Type == "Wrap" or reward_info.Type == "Charm" or reward_info.Type == "Finisher" then
            states.skinchanger_state.fake_owned[reward_name] = true
        end
    elseif modules.ShopLibrary and modules.ShopLibrary.Weapons and modules.ShopLibrary.Weapons[reward_name] then
        local owned = {}
        for _, weapon_data in pairs(states.skinchanger_state.fake_weapon_owned) do
            if type(weapon_data) == "table" and weapon_data.Name then
                owned[weapon_data.Name] = true
            end
        end
        if not owned[reward_name] then
            table.insert(states.skinchanger_state.fake_weapon_owned, {
                Name = reward_name,
                Level = 1,
                XP = 0,
                IsFavorited = false,
                Skin = nil,
            })
        end
        pcall(function()
            if modules.PlayerDataController and modules.PlayerDataController.CurrentData then
                modules.PlayerDataController.CurrentData:Replicate("WeaponInventory")
            end
        end)
    elseif modules.CosmeticLibrary and modules.CosmeticLibrary.Rewards and modules.CosmeticLibrary.Rewards[reward_name] then
        local reward_info = modules.CosmeticLibrary.Rewards[reward_name]
        local reward_entry
        if reward_info.Type == "Skin" then
            if not reward_weapon or reward_weapon == "" then return end
            reward_entry = { Name = reward_name, Quantity = reward_amount, Weapon = reward_weapon }
        else
            reward_entry = { Name = reward_name, Quantity = reward_amount }
        end
        local found = false
        for _, entry in ipairs(states.inventory_state.fake_owned) do
            if entry.Name == reward_entry.Name and entry.Weapon == reward_entry.Weapon and entry.Quantity < 99 then
                entry.Quantity = (entry.Quantity or 1) + (reward_entry.Quantity or 1)
                found = true
                break
            end
        end
        if not found then
            table.insert(states.inventory_state.fake_owned, reward_entry)
        end
        pcall(function()
            if modules.PlayerDataController and modules.PlayerDataController.CurrentData then
                modules.PlayerDataController.CurrentData:Replicate("UnclaimedRewards")
            end
        end)
    end
end

local product_type = "product"
local marketplace_asset_id = 0
local function UpdateMarketplace()
    if not states.spoofer_state.marketplace.fake_robux then return end

    local foundation_overlay = FindContainerWithRobuxPrice()
    if not foundation_overlay or not foundation_overlay:FindFirstChild("SafeAreaFrame") then
        return
    end

    if not marketplace_asset_id or marketplace_asset_id == 0 then
        return
    end

    local robux_text = string.format(
        '<font family="rbxasset://LuaPackages/Packages/_Index/BuilderIcons/BuilderIcons/BuilderIcons.json" weight="400">robux</font> %s',
        FormatNumber(states.spoofer_state.marketplace.robux_amount)
    )

    local info_type = product_type == "gamepass" and Enum.InfoType.GamePass or product_type == "product" and Enum.InfoType.Product or Enum.InfoType.Asset
    local product_info = MarketplaceService:GetProductInfo(marketplace_asset_id, info_type)
    local has_enough = product_info.PriceInRobux <= states.spoofer_state.marketplace.robux_amount

    local BuyButton
    local PromptTitle
    local RobuxBalance
    local SheetContentContainer

    for _, descendant in ipairs(foundation_overlay:GetDescendants()) do
        local parent = descendant.Parent
        local grandparent = parent and parent.Parent

        if descendant.Name == "RobuxPrice" and descendant:IsA("TextLabel") and grandparent and grandparent.Name == "ModalHeader" then
            descendant.RichText = true
            descendant.Text = robux_text
            RobuxBalance = descendant
        elseif descendant.Name == "RobuxAmount" and descendant:IsA("TextLabel") and grandparent and grandparent.Name == "ModalHeader" then
            descendant.Text = has_enough and "Buy item" or "Buy Robux and item"
            PromptTitle = descendant
        elseif descendant:IsA("ImageButton") and parent and parent.Name == "1" then
            BuyButton = descendant
        elseif descendant.Name == "SheetContentContainer" and descendant:IsA("Frame") and parent and parent.Name == "ScrollingFrame" then
            SheetContentContainer = descendant
        elseif descendant.Name == "UpsellSuggestions" and descendant:IsA("Frame") and parent and parent.Name == "SheetContentContainer" and has_enough then
            descendant:Destroy()
        elseif descendant.Name == "DisclosureTerms" and descendant:IsA("Frame") and parent and parent.Name == "1" and has_enough then
            descendant:Destroy()
        end
    end

    if BuyButton then
        local OverlayButton = foundation_overlay:FindFirstChild("nigga")

        if OverlayButton then
            return
        end

        OverlayButton = Instance.new("TextButton")
        OverlayButton.Name = "nigga"
        OverlayButton.BackgroundTransparency = 1
        OverlayButton.Text = ""
        OverlayButton.AutoButtonColor = false
        OverlayButton.ZIndex = BuyButton.ZIndex + 100
        OverlayButton.Size = UDim2.fromOffset(BuyButton.AbsoluteSize.X, BuyButton.AbsoluteSize.Y)
        OverlayButton.Position = UDim2.fromOffset(BuyButton.AbsolutePosition.X, BuyButton.AbsolutePosition.Y + BuyButton.AbsoluteSize.Y + (BuyButton.AbsoluteSize.Y / 2))
        OverlayButton.AnchorPoint = BuyButton.AnchorPoint
        OverlayButton.Parent = foundation_overlay

        BuyButton.Destroying:Connect(function()
            OverlayButton:Destroy()
            product_type = "product"
            marketplace_asset_id = 0
        end)

        -- OPT: no Heartbeat cada frame para overlay
        task.spawn(function()
            while OverlayButton and OverlayButton.Parent and BuyButton and BuyButton.Parent do
                pcall(function()
                    OverlayButton.Position = UDim2.fromOffset(BuyButton.AbsolutePosition.X, BuyButton.AbsoluteSize.Y and (BuyButton.AbsolutePosition.Y + BuyButton.AbsoluteSize.Y + BuyButton.AbsoluteSize.Y / 2) or BuyButton.AbsolutePosition.Y)
                end)
                task.wait(0.2)
            end
        end)

        task.wait(3)

        local purchased = false
        OverlayButton.MouseButton1Click:Connect(function()
            local current_id = getthreadidentity()
            setthreadidentity(2)

            if purchased then
                GuiService:InspectPlayerFromUserId(LocalPlayer.UserId)
                GuiService:CloseInspectMenu()
                return
            end

            local currency_rewards = {}

            for _, bundle in pairs(modules.MonetizationLibrary.Bundles) do
                if bundle.ProductID == marketplace_asset_id or bundle.GamepassName == modules.MonetizationLibrary:GetGamepassName(marketplace_asset_id) then
                    currency_rewards = bundle.Rewards
                    break
                end
            end

            setthreadidentity(current_id)

            local original_bgcolor = BuyButton.BackgroundColor3
            BuyButton.BackgroundColor3 = Color3.fromRGB(38, 61, 143)

            local bgcolor_conn = BuyButton:GetPropertyChangedSignal("BackgroundColor3"):Connect(function()
                BuyButton.BackgroundColor3 = Color3.fromRGB(38, 61, 143)
            end)

            task.wait(2)

            bgcolor_conn:Disconnect()
            BuyButton.BackgroundColor3 = original_bgcolor

            if product_type == "gamepass" then
                MarketplaceService:SignalPromptGamePassPurchaseFinished(LocalPlayer, marketplace_asset_id, has_enough)
            elseif product_type == "product" then
                MarketplaceService:SignalPromptProductPurchaseFinished(LocalPlayer.UserId, marketplace_asset_id, has_enough)
            elseif product_type == "asset" then
                MarketplaceService:SignalPromptPurchaseFinished(LocalPlayer, marketplace_asset_id, has_enough)
            end

            PromptTitle.Text = "Purchase completed"
            PromptTitle.Name = "Title"
            RobuxBalance:Destroy()

            local BodyContainer = Instance.new("Frame")
            BodyContainer.Name = "Body"
            BodyContainer.Parent = SheetContentContainer.Parent
            BodyContainer.AutomaticSize = Enum.AutomaticSize.Y
            BodyContainer.BackgroundTransparency = 1
            BodyContainer.BorderSizePixel = 0
            BodyContainer.LayoutOrder = 1
            BodyContainer.Size = UDim2.fromScale(1, 0)
            BodyContainer.AutoLocalize = false
            SheetContentContainer:Destroy()

            local UIListLayout = Instance.new("UIListLayout")
            UIListLayout.Parent = BodyContainer
            UIListLayout.Padding = UDim.new(0, 16)
            UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
            UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
            UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Center

            local UIPadding = Instance.new("UIPadding")
            UIPadding.Parent = BodyContainer
            UIPadding.PaddingTop = UDim.new(0, 8)

            local Icon = Instance.new("TextLabel")
            Icon.Name = "Icon"
            Icon.Parent = BodyContainer
            Icon.BackgroundTransparency = 1
            Icon.BorderSizePixel = 0
            Icon.LayoutOrder = 0
            Icon.Size = UDim2.fromOffset(48, 48)
            Icon.AutoLocalize = false
            Icon.FontFace = Font.new("rbxasset://LuaPackages/Packages/_Index/BuilderIcons/BuilderIcons/BuilderIcons.json", Enum.FontWeight.Regular)
            Icon.Text = "circle-check"
            Icon.TextColor3 = Color3.fromRGB(213, 215, 221)
            Icon.TextSize = 48

            local Message = Instance.new("TextLabel")
            Message.Name = "Message"
            Message.Parent = BodyContainer
            Message.AutomaticSize = Enum.AutomaticSize.XY
            Message.BackgroundTransparency = 1
            Message.BorderSizePixel = 0
            Message.Size = UDim2.new(0, 0, 0, 0)
            Message.AutoLocalize = false
            Message.FontFace = Font.new("rbxasset://fonts/families/LegacyArimo.json", Enum.FontWeight.Regular)
            Message.Text = "You have successfully bought " .. product_info.Name .. "."
            Message.TextColor3 = Color3.fromRGB(213, 215, 221)
            Message.TextSize = 15
            Message.TextWrapped = true
            Message.TextXAlignment = Enum.TextXAlignment.Left

            local label = BuyButton:FindFirstChildOfClass("TextLabel")
            if label then
                local clone = label:Clone()
                clone.Text = "OK"
                clone.Parent = label.Parent
                label:Destroy()
            end

            setthreadidentity(2)

            task.wait()
            modules.MonetizationController:_FinishPrompt(has_enough)

            if has_enough and product_type == "gamepass" then
                AddGamepass(marketplace_asset_id)
            end

            for _, reward in ipairs(currency_rewards) do
                AddReward(reward.Type or reward.Name, reward.Quantity or 1, reward.Weapon)
            end

            if has_enough and modules.MonetizationLibrary.Gifts[tostring(marketplace_asset_id)] and states.spoofer_state.marketplace.fake_gifting then
                local marketplace = states.spoofer_state.marketplace

                marketplace.fake_gift_robux_spent += product_info.PriceInRobux

                while true do
                    local required = modules.MonetizationLibrary:GetGiftRewardRobuxSpentRequirement(
                        marketplace.fake_gift_rewards_claimed
                    )
                    if marketplace.fake_gift_robux_spent < required then
                        break
                    end
                    marketplace.fake_gift_robux_spent -= required
                    marketplace.fake_gift_rewards_claimed += 1
                end

                modules.PlayerDataController.CurrentData:Replicate("GiftRobuxSpentProgress")
                modules.PlayerDataController.CurrentData:Replicate("GiftRobuxSpentRewardsClaimed")
            end

            purchased = true
        end)
    end
end

CoreGui.DescendantAdded:Connect(UpdateMarketplace)

task.spawn(function()
    pcall(function()
        local original = modules.MonetizationController.PromptProductPurchase

        modules.MonetizationController.PromptProductPurchase = function(self, product_id, ...)
            if product_id and product_id ~= 0 then
                marketplace_asset_id = product_id
                product_type = "product"
            end

            local results = {original(self, product_id, ...)}
            return table.unpack(results)
        end
    end)

    pcall(function()
        local original = modules.MonetizationController.PromptGamePassPurchase

        modules.MonetizationController.PromptGamePassPurchase = function(self, gamepass_id, ...)
            if gamepass_id and gamepass_id ~= 0 then
                marketplace_asset_id = gamepass_id
                product_type = "gamepass"
            end

            local results = {original(self, gamepass_id, ...)}
            return table.unpack(results)
        end
    end)

    pcall(function()
        local original = modules.MonetizationController.PromptPurchase

        modules.MonetizationController.PromptPurchase = function(self, asset_id)
            if asset_id and asset_id ~= 0 then
                marketplace_asset_id = asset_id
                product_type = "asset"
            end

            local results = {original(self, asset_id)}
            return table.unpack(results)
        end
    end)

    pcall(function()
        local remote = ReplicatedStorage.Remotes.Misc.PromptGiftProductPurchase

        NamecallDispatcher:Register(function(self, method, ...)
            if method ~= "InvokeServer" or self ~= remote then
                return false
            end

            local args = { ... }

            if args[1] and args[1] ~= 0 then
                marketplace_asset_id = args[1]
                product_type = "product"
            end

            return NamecallDispatcher:CallOriginal(self, ...)
        end)
    end)
end)

local kCosmeticTypes = {"Skin", "Wrap", "Charm", "Finisher"}
local kCosmeticRarities = {"Common", "Rare", "Legendary", "Mythical", "Unique", "Unobtainable"}

local function GetCosmeticsByRarity(rarity)
    local results = {}
    if not modules.CosmeticLibrary or not modules.CosmeticLibrary.Cosmetics then return results end
    for name, cosmetic in pairs(modules.CosmeticLibrary.Cosmetics) do
        if cosmetic.Rarity == rarity then table.insert(results, name) end
    end
    return results
end

local function GetCosmeticsByType(type_name)
    local results = {}
    if not modules.CosmeticLibrary or not modules.CosmeticLibrary.Cosmetics then return results end
    for name, cosmetic in pairs(modules.CosmeticLibrary.Cosmetics) do
        if cosmetic.Type == type_name then table.insert(results, name) end
    end
    return results
end

local function GetSpecificCosmetic(type_name, cos_name, weapon_name)
    local result
    if not modules.CosmeticLibrary or not modules.CosmeticLibrary.Cosmetics then return result end
    for name, cosmetic in pairs(modules.CosmeticLibrary.Cosmetics) do
        if type_name == "Skin" then
            if name == cos_name and cosmetic.Type == type_name and cosmetic.ItemName == weapon_name then
                result = name
            end
        else
            if name == cos_name and cosmetic.Type == type_name then
                result = name
            end
        end
    end
    return result
end

local function GetAllCosmeticsOfWeapon(weapon_name)
    local results = {}
    if not modules.CosmeticLibrary or not modules.CosmeticLibrary.Cosmetics then return results end
    for name, cosmetic in pairs(modules.CosmeticLibrary.Cosmetics) do
        if cosmetic.Type == "Skin" then
            if cosmetic.ItemName == weapon_name then table.insert(results, name) end
        else
            table.insert(results, name)
        end
    end
    return results
end

local function GetAllCosmetics()
    local results = {}
    if not modules.CosmeticLibrary or not modules.CosmeticLibrary.Cosmetics then return results end
    for name, _ in pairs(modules.CosmeticLibrary.Cosmetics) do
        table.insert(results, name)
    end
    return results
end

local function UnlockSelectedRarity()
    local list = GetCosmeticsByRarity(states.skinchanger_state.general_unlocker.unlock_rarity)
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = true
    end
end

local function LockSelectedRarity()
    local list = GetCosmeticsByRarity(states.skinchanger_state.general_unlocker.unlock_rarity)
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = nil
    end
end

local function UnlockAllOfType()
    local list = GetCosmeticsByType(states.skinchanger_state.general_unlocker.unlock_type)
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = true
    end
end

local function LockAllOfType()
    local list = GetCosmeticsByType(states.skinchanger_state.general_unlocker.unlock_type)
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = nil
    end
end

local function UnlockSpecific()
    local unlocker = states.skinchanger_state.specific_unlocker
    local cosmetic = GetSpecificCosmetic(unlocker.unlock_type, unlocker.cosmetic_name, unlocker.weapon_name)
    if cosmetic and not cosmetic:find("MISSING_") then
        states.skinchanger_state.fake_owned[cosmetic] = true
    end
end

local function LockSpecific()
    local unlocker = states.skinchanger_state.specific_unlocker
    local cosmetic = GetSpecificCosmetic(unlocker.unlock_type, unlocker.cosmetic_name, unlocker.weapon_name)
    if cosmetic and not cosmetic:find("MISSING_") then
        states.skinchanger_state.fake_owned[cosmetic] = nil
    end
end

local function UnlockSpecificWeapon()
    local weapon_name = states.skinchanger_state.specific_unlocker.weapon_name
    local current_data = modules.PlayerDataController and modules.PlayerDataController.CurrentData
    if not current_data then return end
    if weapon_name == nil or weapon_name == "" then return end

    local inventory = current_data:Get("WeaponInventory")
    local owned = {}
    if type(inventory) == "table" then
        for _, weapon_data in pairs(inventory) do
            if type(weapon_data) == "table" and weapon_data.Name then
                owned[weapon_data.Name] = true
            end
        end
    end

    if not owned[weapon_name] then
        table.insert(states.skinchanger_state.fake_weapon_owned, {
            Name = weapon_name,
            Level = 1,
            XP = 0,
            IsFavorited = false,
            Skin = nil,
        })
        states.skinchanger_state.fake_weapon_names[weapon_name] = true
    end
end

local function LockSpecificWeapon()
    local weapon_name = states.skinchanger_state.specific_unlocker.weapon_name
    if not states.skinchanger_state.fake_weapon_names[weapon_name] then return end
    local fake = states.skinchanger_state.fake_weapon_owned
    for i = #fake, 1, -1 do
        if fake[i].Name == weapon_name then
            table.remove(fake, i)
            states.skinchanger_state.fake_weapon_names[weapon_name] = nil
            break
        end
    end
end

local function UnlockAllWeapons()
    local current_data = modules.PlayerDataController and modules.PlayerDataController.CurrentData
    if not current_data then return end

    local inventory = current_data:Get("WeaponInventory")
    local owned = {}
    if type(inventory) == "table" then
        for _, weapon_data in pairs(inventory) do
            if type(weapon_data) == "table" and weapon_data.Name then
                owned[weapon_data.Name] = true
            end
        end
    end

    if not modules.ShopLibrary or not modules.ShopLibrary.GetReleasedOwnableWeapons then return end
    local ownables = modules.ShopLibrary:GetReleasedOwnableWeapons()
    for _, weapon_name in pairs(ownables) do
        if not owned[weapon_name] and not states.skinchanger_state.fake_weapon_names[weapon_name] then
            table.insert(states.skinchanger_state.fake_weapon_owned, {
                Name = weapon_name,
                Level = 1,
                XP = 0,
                IsFavorited = false,
                Skin = nil,
            })
            states.skinchanger_state.fake_weapon_names[weapon_name] = true
        end
    end
end

local function LockAllWeapons()
    local fake = states.skinchanger_state.fake_weapon_owned
    local names = states.skinchanger_state.fake_weapon_names
    for i = #fake, 1, -1 do
        if names[fake[i].Name] then
            table.remove(fake, i)
        end
    end
    table.clear(names)
end

local function UnlockAllForWeapon()
    local weapon_name = states.skinchanger_state.specific_unlocker.weapon_name
    local list = GetAllCosmeticsOfWeapon(weapon_name)
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = true
    end
end

local function LockAllForWeapon()
    local weapon_name = states.skinchanger_state.specific_unlocker.weapon_name
    local list = GetAllCosmeticsOfWeapon(weapon_name)
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = nil
    end
end

local function UnlockAll()
    local list = GetAllCosmetics()
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = true
    end
end

local function LockAll()
    local list = GetAllCosmetics()
    for _, cosmetic in ipairs(list) do
        if cosmetic:find("MISSING_") then continue end
        states.skinchanger_state.fake_owned[cosmetic] = nil
    end
end

local function EquipApply()
    local EquipCosmeticRemote = ReplicatedStorage.Remotes.Data.EquipCosmetic
    local unlocker = states.skinchanger_state.equip_unlocker
    local unlock_type = unlocker.unlock_type
    local cosmetic_name = unlocker.cosmetic_name
    local weapon_name = unlocker.weapon_name
    local is_inverted = unlocker.inverted

    if typeof(weapon_name) ~= "string" or weapon_name == "" then return end

    local name_to_send = if cosmetic_name ~= nil and cosmetic_name ~= "" then cosmetic_name else nil

    local options = {}
    if unlock_type == "Wrap" then
        options.IsInverted = if is_inverted then true else nil
    end

    EquipCosmeticRemote:FireServer(weapon_name, unlock_type, name_to_send, options)
end

local function EquipApplyAll()
    local EquipCosmeticRemote = ReplicatedStorage.Remotes.Data.EquipCosmetic
    local unlocker = states.skinchanger_state.equip_unlocker
    local unlock_type = unlocker.unlock_type
    local cosmetic_name = unlocker.cosmetic_name
    local is_inverted = unlocker.inverted

    if unlock_type == "Skin" then return end
    if not modules.ItemLibrary or not modules.ItemLibrary.Items then return end

    for weapon_name, weapon_data in pairs(modules.ItemLibrary.Items) do
        if unlock_type == "Finisher" and not weapon_data.CanEliminate then
            continue
        end
        if typeof(weapon_name) ~= "string" or weapon_name == "" then
            continue
        end

        local name_to_send = if cosmetic_name ~= nil and cosmetic_name ~= "" then cosmetic_name else nil

        local options = {}
        if unlock_type == "Wrap" then
            options.IsInverted = if is_inverted then true else nil
        end

        EquipCosmeticRemote:FireServer(weapon_name, unlock_type, name_to_send, options)
    end
end

local SkyboxPresets = {
    ["None"] = {SkyboxBk="rbxassetid://14147881792",SkyboxDn="rbxassetid://14147882149",SkyboxFt="rbxassetid://14147882761",SkyboxLf="rbxassetid://14147883091",SkyboxRt="rbxassetid://14147882405",SkyboxUp="rbxassetid://14147881297"},
    Aurora = {SkyboxBk="rbxassetid://116533337330584",SkyboxDn="rbxassetid://80054106187171",SkyboxFt="rbxassetid://94459139270943",SkyboxLf="rbxassetid://116368999680791",SkyboxRt="rbxassetid://125758104196312",SkyboxUp="rbxassetid://107060226443967"},
    Battlerock = {SkyboxBk="rbxassetid://131136284306917",SkyboxDn="rbxassetid://89505977207531",SkyboxFt="rbxassetid://140099243548102",SkyboxLf="rbxassetid://121676169821100",SkyboxRt="rbxassetid://97183886241447",SkyboxUp="rbxassetid://107128620201556"},
    ["Beach Bowl"] = {SkyboxBk="rbxassetid://81804134601271",SkyboxDn="rbxassetid://92395364196932",SkyboxFt="rbxassetid://81804134601271",SkyboxLf="rbxassetid://81804134601271",SkyboxRt="rbxassetid://81804134601271",SkyboxUp="rbxassetid://119089964803065"},
    ["Buoy Base"] = {SkyboxBk="rbxassetid://135017685421888",SkyboxDn="rbxassetid://76610044495625",SkyboxFt="rbxassetid://89675413438577",SkyboxLf="rbxassetid://138307087837279",SkyboxRt="rbxassetid://136723547010707",SkyboxUp="rbxassetid://83404878914838"},
    Clockwork = {SkyboxBk="rbxassetid://86284761193226",SkyboxDn="rbxassetid://111425663631622",SkyboxFt="rbxassetid://115606366886873",SkyboxLf="rbxassetid://127287488325060",SkyboxRt="rbxassetid://126844150113423",SkyboxUp="rbxassetid://74510789204352"},
    ["Dark Matter"] = {SkyboxBk="rbxassetid://97629693450922",SkyboxDn="rbxassetid://97898396690232",SkyboxFt="rbxassetid://134755033418084",SkyboxLf="rbxassetid://118219143707956",SkyboxRt="rbxassetid://114940065588775",SkyboxUp="rbxassetid://95430908943263"},
    ["Flash Black"] = {SkyboxBk="rbxassetid://78426835654353",SkyboxDn="rbxassetid://6213218651",SkyboxFt="rbxassetid://71970982976722",SkyboxLf="rbxassetid://78426835654353",SkyboxRt="rbxassetid://78426835654353",SkyboxUp="rbxassetid://138004866371717"},
    Ghostly = {SkyboxBk="rbxassetid://111506743048183",SkyboxDn="rbxassetid://86198196348228",SkyboxFt="rbxassetid://86265514167302",SkyboxLf="rbxassetid://100257959405445",SkyboxRt="rbxassetid://71935101953120",SkyboxUp="rbxassetid://132011089223498"},
    ["Good Egg"] = {SkyboxBk="rbxassetid://94681381933012",SkyboxDn="rbxassetid://75843838469806",SkyboxFt="rbxassetid://97891957473259",SkyboxLf="rbxassetid://102971518965494",SkyboxRt="rbxassetid://94588890960775",SkyboxUp="rbxassetid://127368871569815"},
    ["Melty Molten"] = {SkyboxBk="rbxassetid://131463907527649",SkyboxDn="rbxassetid://116154164311420",SkyboxFt="rbxassetid://113077689016278",SkyboxLf="rbxassetid://79984367513909",SkyboxRt="rbxassetid://82395195737484",SkyboxUp="rbxassetid://117530106700350"},
    Shiverburn = {SkyboxBk="rbxassetid://113636030839991",SkyboxDn="rbxassetid://118027268179499",SkyboxFt="rbxassetid://76405010847029",SkyboxLf="rbxassetid://112044353352688",SkyboxRt="rbxassetid://121078604572355",SkyboxUp="rbxassetid://132486295432727"},
    ["Spin Dig"] = {SkyboxBk="rbxassetid://124211111366754",SkyboxDn="rbxassetid://120491795220431",SkyboxFt="rbxassetid://130119279111055",SkyboxLf="rbxassetid://70742671331562",SkyboxRt="rbxassetid://76516826791940",SkyboxUp="rbxassetid://100229310567751"},
    ["Sweet Mystery"] = {SkyboxBk="rbxassetid://107264897520277",SkyboxDn="rbxassetid://135637946277638",SkyboxFt="rbxassetid://135705252786048",SkyboxLf="rbxassetid://119667604517747",SkyboxRt="rbxassetid://75904303027092",SkyboxUp="rbxassetid://97011146822716"},
    ["Terrace Dome"] = {SkyboxBk="rbxassetid://98684100016510",SkyboxDn="rbxassetid://108354040356521",SkyboxFt="rbxassetid://95723629635852",SkyboxLf="rbxassetid://106269064939837",SkyboxRt="rbxassetid://136234415079744",SkyboxUp="rbxassetid://75385962780878"},
}
local SkyboxNames = {}
for name in pairs(SkyboxPresets) do SkyboxNames[#SkyboxNames+1] = name end
table.sort(SkyboxNames)

local skyboxData = {enabled=false, selected="None", sky_obj=nil, auto_rotate=false, rotate_speed=10, star_count=3000, sun_size=20, moon_size=11}
local skyboxConn = nil

local function EnsureSky()
    if skyboxData.sky_obj and skyboxData.sky_obj.Parent then return skyboxData.sky_obj end
    local original = Lighting:FindFirstChildOfClass("Sky")
    if original then
        skyboxData.sky_obj = original:Clone()
        skyboxData.sky_obj.Parent = original.Parent
        original:Destroy()
    else
        skyboxData.sky_obj = Instance.new("Sky")
        skyboxData.sky_obj.Parent = Lighting
    end
    return skyboxData.sky_obj
end

local function ApplySkybox(force)
    if not skyboxData.enabled then return end
    local sky = skyboxData.sky_obj
    local needRebuild = force or not sky or not sky.Parent or sky.Parent ~= Lighting
    if needRebuild then
        for _, c in ipairs(Lighting:GetChildren()) do
            if c:IsA("Sky") and not c:GetAttribute("ShakoSky") and not c:GetAttribute("WX_Custom") then
                pcall(function() c:Destroy() end)
            end
        end
        skyboxData.sky_obj = nil
        sky = EnsureSky()
    end
    if not sky then return end
    pcall(function() sky:SetAttribute("ShakoSky", true) end)
    local name = skyboxData.selected
    if (not name or name == "None" or name == "Off") and config and config.SkyboxPreset then
        name = config.SkyboxPreset
    end
    local tex = SkyboxPresets[name] or SkyboxPresets[skyboxData.selected] or SkyboxPresets["None"]
    if not tex then return end
    -- solo escribir si cambió (menos trabajo en Lighting)
    if force or sky.SkyboxBk ~= tex.SkyboxBk then
        sky.SkyboxBk = tex.SkyboxBk
        sky.SkyboxDn = tex.SkyboxDn
        sky.SkyboxFt = tex.SkyboxFt
        sky.SkyboxLf = tex.SkyboxLf
        sky.SkyboxRt = tex.SkyboxRt
        sky.SkyboxUp = tex.SkyboxUp
    end
    sky.SunAngularSize = skyboxData.sun_size or 20
    sky.MoonAngularSize = skyboxData.moon_size or 11
    sky.StarCount = skyboxData.star_count or 3000
    skyboxData.sky_obj = sky
end

skyboxData = skyboxData or {enabled=false, selected="None", sky_obj=nil, auto_rotate=false, rotate_speed=10, star_count=3000, sun_size=20, moon_size=11}
skyboxConn = skyboxConn or nil

local function StartSkyRotate(on)
    if skyboxConn then pcall(function() skyboxConn:Disconnect() end) skyboxConn = nil end
    skyboxData.auto_rotate = on and true or false
    if not on then return end
    skyboxConn = RunService.RenderStepped:Connect(function(dt)
        if not skyboxData.enabled or not skyboxData.auto_rotate then return end
        local sky = EnsureSky()
        if sky then
            sky.SkyboxOrientation = Vector3.new(0, (sky.SkyboxOrientation.Y + (skyboxData.rotate_speed or 10) * dt) % 360, 0)
        end
    end)
end
getgenv()._ShakoStartSkyRotate = StartSkyRotate


states.ratio_state = states.ratio_state or {enabled = false, width = 100, height = 100}
local function updateRatio()
    local s = states.ratio_state
    if not s or not s.enabled then return end
    local cam = workspace.CurrentCamera
    if not cam then return end
    local w = math.clamp(s.width or 100, 0, 150) / 100
    local h = math.clamp(s.height or 100, 0, 150) / 100
    local cf = cam.CFrame
    cam.CFrame = CFrame.fromMatrix(cf.Position, cf.RightVector, cf.UpVector * h, -cf.LookVector * (w / math.max(h, 0.01)))
end
task.spawn(function()
    while true do
        task.wait(0.12)
        if states and states.ratio_state and states.ratio_state.enabled then
            pcall(updateRatio)
        end
    end
end)

-- EXPORT
getgenv()._ShakoStates = states
getgenv()._ShakoModules = modules
getgenv().RivalsSpoof = states.spoofer_state
getgenv().RivalsMods = modules
getgenv()._ShakoUpdateLB = UpdateLeaderboardOptions
getgenv()._ShakoUpdateCharm = UpdateCharmOptions
getgenv()._ShakoUpdateCurrency = UpdateCurrencyOptions
getgenv()._ShakoUpdateDuel = UpdateDuelHistory
getgenv()._ShakoSpoofAvatar = SpoofAvatar
getgenv()._ShakoSetAvatar = SetAvatar
getgenv()._ShakoAnon = AnonymousMode
getgenv()._ShakoBadges = UpdateBadgesOptions
getgenv()._ShakoRefreshNames = RefreshAllNameSpoofs
getgenv()._ShakoRefreshThumbs = RefreshAllThumbnailSpoofs
getgenv()._ShakoUnlockAll = UnlockAll
    getgenv()._ShakoLockAll = LockAll
    getgenv()._ShakoUnlockAllOfType = UnlockAllOfType
    getgenv()._ShakoLockAllOfType = LockAllOfType
    getgenv()._ShakoUnlockRarity = UnlockSelectedRarity
    getgenv()._ShakoLockRarity = LockSelectedRarity
    getgenv()._ShakoUnlockSpecific = UnlockSpecific
    getgenv()._ShakoLockSpecific = LockSpecific
    getgenv()._ShakoUnlockSpecificWeapon = UnlockSpecificWeapon
    getgenv()._ShakoLockSpecificWeapon = LockSpecificWeapon
    getgenv()._ShakoUnlockAllWeapons = UnlockAllWeapons
    getgenv()._ShakoLockAllWeapons = LockAllWeapons
    getgenv()._ShakoUnlockAllForWeapon = UnlockAllForWeapon
    getgenv()._ShakoLockAllForWeapon = LockAllForWeapon
    getgenv()._ShakoEquipApply = EquipApply
    getgenv()._ShakoEquipApplyAll = EquipApplyAll
getgenv()._ShakoLockAll = LockAll
getgenv()._ShakoSpoofDevice = SpoofDevice
getgenv()._ShakoSetDevice = SetDevice
getgenv()._ShakoDeviceSpam = DeviceSpam
getgenv()._ShakoUpdateMarketplace = UpdateMarketplace
getgenv()._ShakoSkyPresets = SkyboxPresets
getgenv()._ShakoApplySkybox = ApplySkybox
-- Persistencia liviana (sin micro-lag): solo repara si se perdió el sky/weather
getgenv().ShakoVisualPersist = true
do
    local lastSkyFix, lastWxFix = 0, 0
    pcall(function()
        Lighting.ChildAdded:Connect(function(c)
            if not c:IsA("Sky") then return end
            if c:GetAttribute("ShakoSky") or c:GetAttribute("WX_Custom") then return end
            if not (skyboxData and skyboxData.enabled) and not (config and config.SkyboxPreset and config.SkyboxPreset ~= "Off") then
                return
            end
            task.defer(function()
                local now = tick()
                if now - lastSkyFix < 0.5 then return end
                lastSkyFix = now
                pcall(function() c:Destroy() end)
                if skyboxData and skyboxData.enabled then
                    pcall(function() ApplySkybox(true) end)
                elseif config and config.SkyboxPreset and config.SkyboxPreset ~= "Off" and Weather and Weather.setSkybox then
                    pcall(function() Weather.setSkybox(config.SkyboxPreset) end)
                end
            end)
        end)
    end)
    task.spawn(function()
        while getgenv().ShakoVisualPersist do
            task.wait(3.5) -- más lento = menos lag
            -- sky solo si falta
            if skyboxData and skyboxData.enabled then
                local sk = skyboxData.sky_obj
                if not sk or not sk.Parent then
                    pcall(function() ApplySkybox(true) end)
                end
            end
            -- weather solo si faltó folder
            if config and (config.WeatherEnabled or config.Weather) then
                local f = workspace:FindFirstChild("_wx")
                if not f and Weather and Weather.enableWeather then
                    local now = tick()
                    if now - lastWxFix > 4 then
                        lastWxFix = now
                        pcall(function() Weather.enableWeather() end)
                    end
                end
            end
            if config and config.SkyboxPreset and config.SkyboxPreset ~= "Off" then
                local has = false
                for _, c in ipairs(Lighting:GetChildren()) do
                    if c:IsA("Sky") and (c:GetAttribute("WX_Custom") or c:GetAttribute("ShakoSky")) then
                        has = true
                        break
                    end
                end
                if not has and Weather and Weather.setSkybox then
                    pcall(function() Weather.setSkybox(config.SkyboxPreset) end)
                end
            end
        end
    end)
    -- respawn: 1 sola reaplicación retardada
    pcall(function()
        LocalPlayer.CharacterAdded:Connect(function()
            task.delay(1.2, function()
                if skyboxData and skyboxData.enabled then pcall(function() ApplySkybox(true) end) end
                local f = workspace:FindFirstChild("_wx")
                if config and (config.WeatherEnabled or config.Weather) and not f then
                    pcall(function() if Weather and Weather.enableWeather then Weather.enableWeather() end end)
                end
            end)
        end)
    end)
end

getgenv()._ShakoEnsureSky = EnsureSky
getgenv()._ShakoSkyData = skyboxData
getgenv()._ShakoUpdateRatio = updateRatio
getgenv()._ShakoSourceOK = true
print("[shako] AURORA spoofers loaded OK")

end)
if not _srcOk then
    warn("[shako] AURORA source error:", _srcErr)
    getgenv()._ShakoSourceOK = false
end

-- ========== SPOOF API (siempre disponible) ==========
RivalsSpoof = getgenv().RivalsSpoof or {
    device="Desktop", old_device="Desktop", spoof_device=false, device_spam=false, spam_rate=1,
    display_name=false, display_name_value="", username=false, username_value="",
    avatar_userid=0, spoof_avatar=false, anonymous_mode=false,
    badges={Premium=false,Verified=false,Influencer=false,Admin=false},
    lb={elo=false,elo_value=2500,streak=false,streak_value=50,kills=false,kills_value=0,wins=false,wins_value=0,level=false,level_value=1},
    leaderboard={ELO=false,elo_value=2500,Streak=false,streak_value=50,Kills=false,kills_value=0,Wins=false,wins_value=0,Level=false,level_value=1},
    charm={charm_rank="Archnemesis",arch_rank=1,s0_charm=false,s1_charm=false,s2_charm=false,s3_charm=false},
    currency={}, marketplace={},
}
-- lb proxy
if RivalsSpoof.leaderboard and not rawget(RivalsSpoof, "lb") then
    local lb = RivalsSpoof.leaderboard
    RivalsSpoof.lb = setmetatable({}, {
        __index = function(_, k)
            local map = {elo="ELO",streak="Streak",kills="Kills",wins="Wins",level="Level"}
            if map[k] then return lb[map[k]] end
            if k:find("_value") then return lb[k] end
            return lb[k]
        end,
        __newindex = function(_, k, v)
            local map = {elo="ELO",streak="Streak",kills="Kills",wins="Wins",level="Level"}
            if map[k] then lb[map[k]] = v
            else lb[k] = v end
        end,
    })
end

local function _st()
    return getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state or RivalsSpoof
end

function RivalsApplyDevice()
    local st = _st()
    if not st then Notify("Device","no state") return end
    local map = {Desktop="MouseKeyboard", Mobile="Touch", Console="Gamepad", VR="VR"}
    local key = st.spoof_device and (st.device or "Desktop") or (st.old_device or "Desktop")
    local ctrl = map[key] or "MouseKeyboard"
    local ok1 = pcall(function()
        game:GetService("ReplicatedStorage").Remotes.Replication.Fighter.SetControls:FireServer(ctrl)
    end)
    pcall(function()
        local m = getgenv()._ShakoModules
        if m and m.ControlsController then
            m.ControlsController.CurrentControls = ctrl
        end
    end)
    pcall(function()
        LocalPlayer:SetAttribute("Controls", ctrl)
        LocalPlayer:SetAttribute("Device", tostring(st.device))
    end)
    if getgenv()._ShakoSpoofDevice then pcall(getgenv()._ShakoSpoofDevice) end
    Notify("Device", string.format("%s → %s (%s)", ok1 and "OK" or "remote?", tostring(st.device), st.spoof_device and "ON" or "OFF"))
end

function RivalsDeviceSpam(on)
    local st = _st()
    if not st then return end
    st.device_spam = on and true or false
    if getgenv()._ShakoDeviceSpam then
        pcall(getgenv()._ShakoDeviceSpam)
    else
        if not on then RivalsApplyDevice() return end
        task.spawn(function()
            local list = {"Desktop","Mobile","Console","VR"}
            while st.device_spam do
                st.device = list[math.random(1,#list)]
                st.spoof_device = true
                RivalsApplyDevice()
                task.wait(st.spam_rate or 1)
            end
        end)
    end
end

function PatchProfileStats()
    -- throttled + ligero (sin escanear todo el arbol cada vez)
    if tick() - (getgenv()._ShakoPatchStatsAt or 0) < 1.5 then return end
    getgenv()._ShakoPatchStatsAt = tick()
    local st = (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state) or (_st and _st())
    if not st or not st.leaderboard then return end
    local lb = st.leaderboard
    if not (lb.Level or lb.Streak or lb.Wins or lb.Kills) then return end
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return end
    local me = LocalPlayer.Name
    local meD = LocalPlayer.DisplayName or ""
    local n = 0
    -- solo mirar ScreenGuis de perfil, no todo PlayerGui
    for _, gui in ipairs(pg:GetChildren()) do
        if not gui:IsA("ScreenGui") and not gui:IsA("LayerCollector") then continue end
        local gname = string.lower(gui.Name or "")
        if gname:find("kill") or gname:find("feed") or gname:find("hud") or gname:find("cross") then
            continue
        end
        for _, obj in ipairs(gui:GetDescendants()) do
            if n > 25 then break end
            if not (obj:IsA("TextLabel") or obj:IsA("TextButton")) then continue end
            local t = obj.Text
            if typeof(t) ~= "string" or not t:match("^%s*[%d%.,]+%s*$") then continue end
            local on = obj.Name or ""
            local pn = obj.Parent and obj.Parent.Name or ""
            local chain = string.lower(on .. " " .. pn)
            -- skip si parent chain tiene nombre ajeno (check rapido 4 niveles)
            local other = false
            local p = obj.Parent
            for _ = 1, 5 do
                if not p then break end
                if p.Name == me or p.Name == meD then break end
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and p.Name == plr.Name then other = true break end
                end
                if other then break end
                p = p.Parent
            end
            if other then continue end
            local want = nil
            if lb.Level and (chain:find("level", 1, true) or (on == "Value" and pn == "Level")) then
                want = tostring(lb.level_value)
            elseif lb.Streak and chain:find("streak", 1, true) then
                want = tostring(lb.streak_value)
            elseif lb.Wins and chain:find("wins", 1, true) and not chain:find("winrate", 1, true) then
                want = tostring(lb.wins_value)
            end
            if want and t ~= want then
                pcall(function() obj.Text = want end)
                n = n + 1
            end
        end
        if n > 25 then break end
    end
end

function RivalsApplyLeaderboard()
    local st = _st()
    if st and st.leaderboard then
        local lb = st.leaderboard
        pcall(function()
            if lb.Level then LocalPlayer:SetAttribute("Level", lb.level_value) end
            if lb.Streak then LocalPlayer:SetAttribute("StatisticDuelsWinStreak", lb.streak_value) end
            if lb.ELO then
                pcall(function() LocalPlayer:SetAttribute("StatisticHighestELO", lb.elo_value) end)
                pcall(function() LocalPlayer:SetAttribute("HighestELO", lb.elo_value) end)
            end
        end)
    end
    if getgenv()._ShakoUpdateLB then
        local ok, err = pcall(getgenv()._ShakoUpdateLB)
        Notify("LB", ok and "applied" or tostring(err))
    else
        Notify("LB", "attrs only")
    end
    task.defer(function()
        task.wait(0.3)
        pcall(PatchProfileStats)
    end)
end

function RivalsUpdateCharm()
    pcall(function()
        if getgenv()._ShakoHookCharm then getgenv()._ShakoHookCharm() end
    end)
    if getgenv()._ShakoUpdateCharm then
        local ok, err = pcall(getgenv()._ShakoUpdateCharm)
        Notify("Rank/Charm", ok and "aplicado (seasons + ELO)" or tostring(err))
    else
        Notify("Rank/Charm", "UpdateCharm no cargado")
    end
end

function RivalsUpdateCurrency()
    if getgenv()._ShakoUpdateCurrency then pcall(getgenv()._ShakoUpdateCurrency)
    else
        pcall(function()
            local m = getgenv()._ShakoModules
            local cd = m and m.PlayerDataController and m.PlayerDataController.CurrentData
            if cd then
                for _, k in ipairs({"WeaponKeys","UnlockTokens","EventCurrency","Glory","SkinTickets"}) do
                    pcall(function() cd:Replicate(k) end)
                end
            end
        end)
    end
    Notify("Currency", "replicated")
end

function RivalsUpdateDuelHistory()
    if getgenv()._ShakoUpdateDuel then pcall(getgenv()._ShakoUpdateDuel) end
end

function RivalsSetAvatarSpoof(on)
    local st = _st()
    if st then st.spoof_avatar = on and true or false end
    if getgenv()._ShakoSpoofAvatar then
        pcall(getgenv()._ShakoSpoofAvatar)
    end
    -- publicar / quitar marca shako para que otros con el script te vean
    task.defer(function()
        if on then
            pcall(function() if ShakoSync then ShakoSync.broadcast() end end)
        else
            pcall(function()
                LocalPlayer:SetAttribute("ShakoWinAvatar", nil)
                LocalPlayer:SetAttribute("ShakoWinUser", nil)
            end)
        end
    end)
    -- stop keep loop if OFF
    if not on then
        if getgenv()._ShakoAvatarKeepThread then
            pcall(function() task.cancel(getgenv()._ShakoAvatarKeepThread) end)
            getgenv()._ShakoAvatarKeepThread = nil
        end
        if getgenv()._ShakoRefreshThumbs then pcall(getgenv()._ShakoRefreshThumbs) end
        return
    end
    if on and st then
        local uid = tonumber(st.avatar_userid) or tonumber(config.SkinTargetUserId) or 0
        if uid > 0 then
            config.SkinTargetUserId = uid
            config.SkinchangerEnabled = true
            local function applyOnce()
                local char = LocalPlayer.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if hum then
                    local ok, desc = pcall(function()
                        return Players:GetHumanoidDescriptionFromUserIdAsync(uid)
                    end)
                    if ok and desc then
                        pcall(function()
                            if hum.ApplyDescriptionReset then
                                hum:ApplyDescriptionReset(desc)
                            else
                                hum:ApplyDescription(desc)
                            end
                        end)
                    end
                end
                if getgenv()._ShakoSetAvatar then
                    pcall(getgenv()._ShakoSetAvatar, false)
                end
                if Skin and Skin.copyAvatar then
                    pcall(function() Skin.copyAvatar(uid) end)
                end
            end
            task.spawn(function()
                applyOnce()
                task.wait(0.5)
                applyOnce()
                task.wait(1.2)
                applyOnce()
            end)
            -- keep loop: cada 2s si faltan ropa/acc (partida / range los quita)
            if getgenv()._ShakoAvatarKeepThread then
                pcall(function() task.cancel(getgenv()._ShakoAvatarKeepThread) end)
            end
            getgenv()._ShakoAvatarKeepThread = task.spawn(function()
                while true do
                    task.wait(2)
                    local s = _st()
                    if not s or not s.spoof_avatar then break end
                    local c = LocalPlayer.Character
                    if not c then continue end
                    local hasShirt = c:FindFirstChildOfClass("Shirt") ~= nil
                    local hasPants = c:FindFirstChildOfClass("Pants") ~= nil
                    local accN = 0
                    for _, v in ipairs(c:GetChildren()) do
                        if v:IsA("Accessory") or v:IsA("Hat") then accN = accN + 1 end
                    end
                    if (not hasShirt) or (not hasPants) or accN < 1 then
                        pcall(applyOnce)
                    end
                end
                getgenv()._ShakoAvatarKeepThread = nil
            end)
        end
    end
    if getgenv()._ShakoRefreshThumbs then pcall(getgenv()._ShakoRefreshThumbs) end
end



-- ==================== SHAKO SYNC: ver avatar spoof de otros con el mismo script ====================
-- Cada cliente publica su avatar UserId; otros clientes shako lo leen y aplican SOLO visual local.
local SHAKO_ATTR = "ShakoWinAvatar"
local SHAKO_TAG  = "ShakoWinUser"
local ShakoSync = {
    enabled = true,
    applied = {}, -- [player] = userId applied
    hl = {},      -- highlight peers
}

local function shakoGetMyAvatarUid()
    local st = (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state) or states and states.spoofer_state
    local uid = 0
    if st and st.spoof_avatar then
        uid = tonumber(st.avatar_userid) or tonumber(config.SkinTargetUserId) or 0
    end
    if uid <= 0 and config.SkinchangerEnabled then
        uid = tonumber(config.SkinTargetUserId) or 0
    end
    return uid
end

function ShakoSync.broadcast()
    if not ShakoSync.enabled then return end
    local uid = shakoGetMyAvatarUid()
    if uid <= 0 then
        pcall(function() LocalPlayer:SetAttribute(SHAKO_ATTR, nil) end)
        pcall(function() LocalPlayer:SetAttribute(SHAKO_TAG, nil) end)
        return
    end
    pcall(function() LocalPlayer:SetAttribute(SHAKO_ATTR, uid) end)
    pcall(function() LocalPlayer:SetAttribute(SHAKO_TAG, true) end)
    local char = LocalPlayer.Character
    if char then
        pcall(function() char:SetAttribute(SHAKO_ATTR, uid) end)
        pcall(function() char:SetAttribute(SHAKO_TAG, true) end)
        -- StringValue marcador (por si el juego replica hijos del character)
        local sv = char:FindFirstChild(SHAKO_ATTR)
        if not sv then
            sv = Instance.new("StringValue")
            sv.Name = SHAKO_ATTR
            sv.Parent = char
        end
        sv.Value = tostring(uid)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            pcall(function() hum:SetAttribute(SHAKO_ATTR, uid) end)
            pcall(function() hum:SetAttribute(SHAKO_TAG, true) end)
        end
    end
end

local function readPeerUid(plr)
    if not plr or plr == LocalPlayer then return nil end
    local uid = tonumber(plr:GetAttribute(SHAKO_ATTR))
    if uid and uid > 0 then return uid end
    local char = plr.Character
    if char then
        uid = tonumber(char:GetAttribute(SHAKO_ATTR))
        if uid and uid > 0 then return uid end
        local sv = char:FindFirstChild(SHAKO_ATTR)
        if sv and sv:IsA("StringValue") then
            uid = tonumber(sv.Value)
            if uid and uid > 0 then return uid end
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            uid = tonumber(hum:GetAttribute(SHAKO_ATTR))
            if uid and uid > 0 then return uid end
        end
    end
    return nil
end

local function isShakoPeer(plr)
    if not plr or plr == LocalPlayer then return false end
    if plr:GetAttribute(SHAKO_TAG) then return true end
    local char = plr.Character
    if char and (char:GetAttribute(SHAKO_TAG) or char:FindFirstChild(SHAKO_ATTR)) then return true end
    return readPeerUid(plr) ~= nil
end

function ShakoSync.applyToOther(plr, userId)
    if not plr or plr == LocalPlayer or not userId or userId <= 0 then return end
    local char = plr.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    -- ya aplicado este id
    if ShakoSync.applied[plr] == userId and char:FindFirstChildOfClass("Shirt") then
        return
    end
    ShakoSync.applied[plr] = userId
    task.spawn(function()
        -- HumanoidDescription local (solo se ve en TU cliente)
        local ok, desc = pcall(function()
            return Players:GetHumanoidDescriptionFromUserIdAsync(userId)
        end)
        if ok and desc then
            pcall(function()
                if hum.ApplyDescriptionReset then
                    hum:ApplyDescriptionReset(desc)
                else
                    hum:ApplyDescription(desc)
                end
            end)
        end
        -- fallback: clonar modelo
        pcall(function()
            local rigOk, rig = pcall(function()
                return Players:CreateHumanoidModelFromUserId(userId)
            end)
            if not rigOk or not rig then return end
            for _, v in ipairs(char:GetChildren()) do
                if v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic")
                    or v:IsA("BodyColors") or v:IsA("CharacterMesh") then
                    pcall(function() v:Destroy() end)
                end
            end
            for _, item in ipairs(rig:GetChildren()) do
                if item:IsA("Accessory") then
                    pcall(function()
                        if hum.AddAccessory then hum:AddAccessory(item:Clone()) else item:Clone().Parent = char end
                    end)
                elseif item:IsA("Shirt") or item:IsA("Pants") or item:IsA("ShirtGraphic") or item:IsA("BodyColors") then
                    pcall(function() item:Clone().Parent = char end)
                end
            end
            pcall(function() rig:Destroy() end)
        end)
        -- highlight peer shako (cyan suave)
        pcall(function()
            if ShakoSync.hl[plr] then ShakoSync.hl[plr]:Destroy() end
            local hl = Instance.new("Highlight")
            hl.Name = "ShakoPeerHL"
            hl.FillColor = Color3.fromRGB(0, 200, 255)
            hl.OutlineColor = Color3.fromRGB(0, 255, 255)
            hl.FillTransparency = 0.85
            hl.OutlineTransparency = 0.35
            hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            hl.Parent = char
            ShakoSync.hl[plr] = hl
        end)
    end)
end

function ShakoSync.scan()
    if not ShakoSync.enabled then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        local uid = readPeerUid(plr)
        if uid then
            ShakoSync.applyToOther(plr, uid)
        end
    end
end

function ShakoSync.start()
    ShakoSync.enabled = true
    if ShakoSync._conn then return end
    ShakoSync._conn = true
    -- broadcast + scan loop
    task.spawn(function()
        while ShakoSync.enabled do
            pcall(ShakoSync.broadcast)
            pcall(ShakoSync.scan)
            task.wait(2)
        end
    end)
    Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function()
            task.wait(1)
            if not ShakoSync.enabled then return end
            local uid = readPeerUid(plr)
            if uid then ShakoSync.applyToOther(plr, uid) end
        end)
        pcall(function()
            plr:GetAttributeChangedSignal(SHAKO_ATTR):Connect(function()
                local uid = readPeerUid(plr)
                if uid then ShakoSync.applyToOther(plr, uid) end
            end)
        end)
    end)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            pcall(function()
                plr:GetAttributeChangedSignal(SHAKO_ATTR):Connect(function()
                    local uid = readPeerUid(plr)
                    if uid then ShakoSync.applyToOther(plr, uid) end
                end)
            end)
            plr.CharacterAdded:Connect(function()
                task.wait(1)
                ShakoSync.applied[plr] = nil -- force reapply on respawn
                local uid = readPeerUid(plr)
                if uid then ShakoSync.applyToOther(plr, uid) end
            end)
        end
    end
    LocalPlayer.CharacterAdded:Connect(function()
        task.wait(0.5)
        pcall(ShakoSync.broadcast)
    end)
end

function ShakoSync.stop()
    ShakoSync.enabled = false
    ShakoSync._conn = false
    pcall(function() LocalPlayer:SetAttribute(SHAKO_ATTR, nil) end)
    pcall(function() LocalPlayer:SetAttribute(SHAKO_TAG, nil) end)
    for plr, hl in pairs(ShakoSync.hl) do
        pcall(function() hl:Destroy() end)
    end
    ShakoSync.hl = {}
    ShakoSync.applied = {}
end

getgenv().ShakoSync = ShakoSync
task.defer(function()
    task.wait(1)
    pcall(ShakoSync.start)
end)


function RivalsSetAvatar(useOld)
    if getgenv()._ShakoSetAvatar then pcall(getgenv()._ShakoSetAvatar, useOld) end
end

function RivalsSetAnonymous(on)
    local st = _st()
    if st then st.anonymous_mode = on and true or false end
    if getgenv()._ShakoAnon then pcall(getgenv()._ShakoAnon) end
    Notify("Anon", on and "ON" or "OFF")
end

function RivalsUpdateBadges()
    local st = _st()
    local b = st and st.badges
    if b then
        pcall(function()
            LocalPlayer:SetAttribute("IsPremium", b.Premium and true or false)
            LocalPlayer:SetAttribute("HasVerifiedBadge", b.Verified and true or false)
            LocalPlayer:SetAttribute("IsInfluencer", b.Influencer and true or false)
            LocalPlayer:SetAttribute("IsRobloxEmployee", b.Admin and true or false)
        end)
    end
    if getgenv()._ShakoBadges then pcall(getgenv()._ShakoBadges) end
    Notify("Badges", "OK")
end

local function ForceHumanoidDisplayName()
    local st = _st()
    if not st or not st.display_name then return end
    local fake = tostring(st.display_name_value or "")
    if fake == "" then return end
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum and hum.DisplayName ~= fake then
        pcall(function() hum.DisplayName = fake end)
    end
end

local _lastPatchCG = 0
local function PatchCoreGuiNames()
    if tick() - _lastPatchCG < 0.8 then return end
    _lastPatchCG = tick()
    local st = _st and _st() or (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state)
    if not st or (not st.display_name and not st.username) then return end
    local fakeDisp = tostring(st.display_name_value or "")
    local fakeUser = tostring(st.username_value or "")
    local realDisp = LocalPlayer.DisplayName
    local realUser = LocalPlayer.Name
    if fakeDisp == "" and fakeUser == "" then return end

    local blockName = {
        Wins=true, Value=true, Streak=true, Level=true, Kills=true,
        Score=true, ELO=true, Amount=true, Count=true, Number=true,
    }

    local function patchObj(obj)
        if not (obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox")) then return end
        local on = obj.Name or ""
        local pn = obj.Parent and obj.Parent.Name or ""
        if blockName[on] or blockName[pn] then return end
        -- solo nombres
        local allow = on == "DisplayName" or on == "PlayerName" or on == "NameLabel"
            or on == "Username" or on == "Handle" or on == "HeaderText" or on == "Title"
            or (pn == "NameContainer") or (pn == "Title") or (pn == "Subtitle")
        local t = obj.Text
        if typeof(t) ~= "string" or t == "" then return end
        -- exact / @ only (no replace dentro de numeros)
        local nt = t
        if st.display_name and fakeDisp ~= "" and realDisp ~= "" and #realDisp >= 2 then
            if t == realDisp then
                nt = fakeDisp
            elseif allow and string.find(t, realDisp, 1, true) and not string.find(t, fakeDisp, 1, true) then
                local i, j = string.find(t, realDisp, 1, true)
                if i then nt = string.sub(t, 1, i - 1) .. fakeDisp .. string.sub(t, j + 1) end
            end
        end
        if st.username and fakeUser ~= "" and realUser ~= "" and #realUser >= 3 then
            if nt == realUser then
                nt = fakeUser
            elseif nt == "@" .. realUser then
                nt = "@" .. fakeUser
            elseif allow and string.find(nt, "@" .. realUser, 1, true) then
                local i, j = string.find(nt, "@" .. realUser, 1, true)
                if i then nt = string.sub(nt, 1, i - 1) .. "@" .. fakeUser .. string.sub(nt, j + 1) end
            end
        end
        if nt ~= t then pcall(function() obj.Text = nt end) end
    end

    pcall(function()
        local cg = game:GetService("CoreGui")
        for _, obj in ipairs(cg:GetDescendants()) do patchObj(obj) end
    end)
    pcall(function()
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg then for _, obj in ipairs(pg:GetDescendants()) do patchObj(obj) end end
    end)
end

function RivalsRefreshNames()
    local st = _st()
    -- sync a config.Spoof* por si otro path lo usa
    pcall(function()
        if st and config then
            if st.display_name then
                config.SpoofEnabled = true
                config.SpoofMyName = true
                config.SpoofMyNameValue = tostring(st.display_name_value or "")
                config.SpoofProfileUI = true
            end
            if st.username then
                config.SpoofMyUserValue = tostring(st.username_value or "")
            end
        end
    end)
    ForceHumanoidDisplayName()
    if getgenv()._ShakoRefreshNames then pcall(getgenv()._ShakoRefreshNames) end
    PatchCoreGuiNames()
    -- Spoof module tree patch
    pcall(function()
        if Spoof and Spoof.patchMyProfileUI then Spoof.patchMyProfileUI() end
    end)
end

-- keep spoofed name on respawn (killfeed usa Humanoid.DisplayName)
if not getgenv()._ShakoNameCharHook then
    getgenv()._ShakoNameCharHook = true
    LocalPlayer.CharacterAdded:Connect(function(char)
        task.defer(function()
            task.wait(0.35)
            ForceHumanoidDisplayName()
            if getgenv()._ShakoRefreshNames then pcall(getgenv()._ShakoRefreshNames) end
            task.wait(0.5)
            ForceHumanoidDisplayName()
            PatchCoreGuiNames()
        end)
    end)
end

-- ESC / pause menu → patch CoreGui player list
if not getgenv()._ShakoNameEscHook then
    getgenv()._ShakoNameEscHook = true
    UserInputService.InputBegan:Connect(function(input, gpe)
        if input.KeyCode == Enum.KeyCode.Escape then
            task.defer(function()
                task.wait(0.15)
                local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
                if st and (st.display_name or st.username) then
                    ForceHumanoidDisplayName()
                    PatchCoreGuiNames()
                    if getgenv()._ShakoRefreshNames then pcall(getgenv()._ShakoRefreshNames) end
                end
            end)
            task.delay(0.4, function()
                local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
                if st and (st.display_name or st.username) then PatchCoreGuiNames() end
            end)
        end
    end)
end

-- loop suave: DisplayName + killfeed labels (cada 0.75s, solo si ON)
if not getgenv()._ShakoNameLoop then
    getgenv()._ShakoNameLoop = true
    task.spawn(function()
        while true do
            task.wait(2.0)
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.display_name then
                pcall(ForceHumanoidDisplayName)
            end
        end
    end)
end

-- CoreGui: solo al añadir labels (ESC list / notifs) — liviano
if not getgenv()._ShakoCoreGuiNameHook then
    getgenv()._ShakoCoreGuiNameHook = true
    pcall(function()
        local cg = game:GetService("CoreGui")
        cg.DescendantAdded:Connect(function(obj)
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if not st or (not st.display_name and not st.username) then return end
            if not (obj:IsA("TextLabel") or obj:IsA("TextButton")) then return end
            -- solo nombres, no cada frame
            local on = obj.Name or ""
            if on ~= "DisplayName" and on ~= "PlayerName" and on ~= "Username" and on ~= "Handle" and on ~= "NameLabel" then return end
            task.defer(function()
                local fakeDisp = tostring(st.display_name_value or "")
                local realDisp = LocalPlayer.DisplayName
                local t = obj.Text
                if typeof(t) == "string" and st.display_name and fakeDisp ~= "" and realDisp ~= ""
                    and string.find(t, realDisp, 1, true) and not string.find(t, fakeDisp, 1, true) then
                    local i, j = string.find(t, realDisp, 1, true)
                    if i then pcall(function() obj.Text = string.sub(t, 1, i - 1) .. fakeDisp .. string.sub(t, j + 1) end) end
                end
                local fakeUser = tostring(st.username_value or "")
                local realUser = LocalPlayer.Name
                t = obj.Text
                if typeof(t) == "string" and st.username and fakeUser ~= "" and realUser ~= ""
                    and string.find(t, realUser, 1, true) and not string.find(t, fakeUser, 1, true) then
                    local i, j = string.find(t, realUser, 1, true)
                    if i then pcall(function() obj.Text = string.sub(t, 1, i - 1) .. fakeUser .. string.sub(t, j + 1) end) end
                end
            end)
        end)
    end)
end

function RivalsRefreshThumbnails()
    if getgenv()._ShakoRefreshThumbs then pcall(getgenv()._ShakoRefreshThumbs) end
end

function RivalsUnlockAll()
    if getgenv()._ShakoUnlockAll then
        local ok, err = pcall(getgenv()._ShakoUnlockAll)
        Notify("Unlock", ok and "ALL OK" or tostring(err))
    else
        Notify("Unlock", "source no cargo")
    end
end

function RivalsLockAll()
    if getgenv()._ShakoLockAll then pcall(getgenv()._ShakoLockAll) end
end

function RivalsUpdateMarketplace()
    if getgenv()._ShakoUpdateMarketplace then pcall(getgenv()._ShakoUpdateMarketplace) end
end

-- sync RivalsSpoof table with states when source loaded
task.defer(function()
    if getgenv().RivalsSpoof then
        RivalsSpoof = getgenv().RivalsSpoof
        -- ensure lb proxy
        if RivalsSpoof.leaderboard and type(RivalsSpoof.lb) ~= "table" then
            local lb = RivalsSpoof.leaderboard
            RivalsSpoof.lb = setmetatable({}, {
                __index = function(_, k)
                    local map = {elo="ELO",streak="Streak",kills="Kills",wins="Wins",level="Level"}
                    if map[k] then return lb[map[k]] end
                    return lb[k]
                end,
                __newindex = function(_, k, v)
                    local map = {elo="ELO",streak="Streak",kills="Kills",wins="Wins",level="Level"}
                    if map[k] then lb[map[k]] = v else lb[k] = v end
                end,
            })
        end
    end
    if getgenv()._ShakoSourceOK then
        Notify("Spoof", "AURORA source OK")
    else
        Notify("Spoof", "source FAIL — fallbacks activos")
    end
end)



getgenv().config = {
    Enabled = false,
    StickHeight = -2.8,
    StickOffsetX = 0,
    StickOffsetZ = 0,
    StickMultiPos = true,
    StickDesync = true, -- no ves el TP (igual que rage)
    StickAway = 6,
    StickPosInterval = 0.18,
    StickFireHead = true,

    MaxRange = 1e9, -- infinito efectivo anti-UE
    TeamCheck = true, -- NUNCA pegarse al team si ON

    PriorityMode = "Closest",
    TargetLock = false,
    FakePosition = true,
    FakePosStrength = 8,
    HitboxExpand = false,
    HitboxSize = 16,

    -- UE Counter (ex Anti Unnamed)
    UECounterEnabled = false,
    UECounterY = 500,
    UECounterCooldown = 0.2,
    UECounterHideDist = 500, -- hide / void height
    UECounterAttackSpeed = 1, -- velocidad ataque/reposición
    AntiUnnamed = false, -- alias de UECounterEnabled (otros sistemas)

    AntiSafeZone = false,
    SafeZoneMinY = 30,
    AntiVoidRescue = false,
    AntiVoidY = 40,

    StrafeEnabled = false,
    OrbitEnabled = false,
    OrbitRange = 10,
    OrbitSpeed = 16,
    StrafeSpeed = 18,

    -- Hit / Kill Songs
    HitSoundEnabled = false,
    FXHitMarker = false,
    FXHitMarkerStyle = "X",
    FXHitMarkerColor = Color3.fromRGB(255, 255, 255),
    FXHitMarkerCritColor = Color3.fromRGB(255, 194, 75),
    FXHitMarkerLethalColor = Color3.fromRGB(255, 64, 78),
    FXHitMarkerGap = 5,
    FXHitMarkerLen = 8,
    FXHitMarkerThickness = 2,
    FXBeamTracer = false,
    FXBeamStyle = "Line",
    FXBeamHitColor = Color3.fromRGB(255, 194, 75),
    FXBeamDuration = 0.35,
    FXBeamThickness = 0.12,
    FXDamageNumbers = false,
    FXKillBanner = false,
    FXKillBannerColor = Color3.fromRGB(255, 194, 75),
    KillSoundEnabled = true,
    HitSoundVolume = 6,
    KillSoundVolume = 7,
    HitSoundMode = "Random",
    KillSoundMode = "Random",
    SelectedHitSound = "Rust HS",
    HitSoundPitch = 1,
    SelectedKillSound = "OOF",
    HitSoundCooldown = 0.08,
    KillSoundCooldown = 0.35,

    TargetName = "",
    UseNameTarget = false,

    FlyEnabled = false,
    FlySpeed = 120,
    FlyBoost = 1.8,
    JumpPowerEnabled = false,
    JumpPower = 100,
    JumpBoostForce = true, -- fuerza Y en Rivals (WalkSpeed/Jump se resetean)
    NoclipEnabled = false,
    AntiFling = true,
    SpeedEnabled = false,
    WalkSpeed = 50,
    SpeedVelocityMode = true,
    DefaultWalkSpeed = 16,
    DefaultJumpPower = 50,
    DefaultJumpHeight = 7.2,
    SlideBoostEnabled = false,
    SlideBoostMult = 2.5,
    SlideBoostKey = "C", -- solo C por defecto
    SlideBaseSpeed = 45, -- velocidad base del slide si Speed OFF
    InfinityJump = false,

    AntiAimEnabled = false,
    AntiAimMode = "Spin",
    SpinSpeed = 25,
    JitterAngle = 45,
    ManualYaw = 180,
    AAPitch = 0,
    AAInvert = false,
    -- Underground AA (pitch down + jitter, NO tp bajo mapa)
    AntiAimUnderground = false,
    AntiAimUndergroundY = -25, -- degrees pitch down (legacy name, now pitch)
    UndergroundPitch = -25,    -- -15 a -50 recomendado
    UndergroundJitter = 35,    -- yaw jitter degrees
    UndergroundRandom = true,  -- random delay/variation
    UndergroundSpin = 0.4,     -- factor de spin suave (0-1)

    SpawnDelay = 1.0,

    RagebotEnabled = false,
    -- Sleepy Counter: TP al lado de la mano (lejitos) + auto head
    SleepyCounterEnabled = false,
    SleepyCounterSideDist = 14,   -- lejos de la mano
    SleepyCounterHandY = 0.5,
    SleepyCounterHeight = 1.2,
    SleepyCounterFireRate = 0.02,
    SleepyCounterTeamCheck = true,
    SleepyCounterRange = 1e12,
    -- Kicia Counter: anti kiciahook rage
    KiciaCounterEnabled = false,
    KiciaCounterSkyY = 450,          -- altura en el cielo
    KiciaCounterLineWidth = 80,      -- ancho de la linea lado a lado
    KiciaCounterMoveSpeed = 400,     -- studs/seg entre puntos (rapido)
    KiciaCounterFireRate = 0.015,
    KiciaCounterTeamCheck = true,
    KiciaCounterShoot = true,        -- seguir disparando a head mientras esta en cielo
    RageIndicatorEnabled = true,
    RageStatusEnabled = true,
    RageIndicatorSnapline = true,
    RageIndicatorColor = "Red",
    RageStatusMode = "static", -- static / crosshair
    RageStatusOffsetY = 48,

    RageDesync = true, -- siempre ON dentro del rage (no UI separada)
    RageHideTP = true,
    RageLookSyncOnShoot = false,
    RageHideLocal = false,
    RagebotHeight = 2.2,
    RagebotRange = math.huge,
    RagebotInfiniteRange = true,
    RagebotTeamCheck = true,
    RagebotForceFFA = false,
 -- true = siempre FFA (ignora team)
    RagebotForceFFA = false,
    RagebotPriorityHP = true,
    RagebotSkipFF = true,
    RagebotLookAtHead = false,
    RagebotOffsetX = 0,
    RagebotOffsetZ = 0,
    RagebotPredict = true,
    RagebotPredictAmount = 0.12,
    -- Rivals UseItem rage
    RageWeaponSlot = "Melee",
    RageFireRate = 0,
    RageAttackSpeed = 10, -- UE-style: que tan rapido se tpea/ataca (1 lento - 20 insta)
    RageDesyncKnifeY = 6,
    RageDesyncNormalY = 1,
    RageDesyncNormalZ = 2,
    RageSkipDeflect = true,
    RageUseItem = true, -- always on with ragebot

    RageShotsPerTick = 3,          -- balas por frame
    RageMultipoint = true,         -- head + upper torso
    RageSwitchDelay = 0.05,        -- al morir target, siguiente
    RageResolver = true,           -- ajusta Y si el target se mueve raro
    RageBodyAimIfWall = false,     -- si head bloqueada usa torso


    VoidRage = false,
    VoidRageAwayDist = 99999999,
    RageVoidFirst = true,
    RageVoidOnKill = true,
    RageVoidHideTime = 0.40, -- min efectivo 0.28
    RageVoidAttackTime = 0.45, -- tiempo atacando

    RagePrioritizeRagers = true,
    VoidRageInterval = 0.35,
    VoidRageMethods = "Side",
    VoidRageReturn = true,

    AimbotEnabled = false,
    AimbotFOV = 400,
    AimbotSmooth = 0.9,
    AimbotPart = "Head",
    AimbotTeamCheck = true,
    AimbotKeyHold = true,

    SilentAimEnabled = false,
    SilentHitPart = "Head",
    SilentPredict = false, -- OFF por defecto
    SilentPredictAmount = 0.06,
    SilentHeadOffsetY = 0.35, -- un poco arriba del centro de head
    SilentFOV = 2000, -- FOV grande anti-UE lejano
    SilentShowFOV = true,
    SilentTeamCheck = true,
    SilentForceAiming = true,
    SilentFOVFill = true,
    SilentFOVFillTransparency = 0.92,
    SilentFOVAnimated = true,
    SilentFOVSpin = true,
    SilentFOVSpinSpeed = 1.2,
    SilentFOVThickness = 1.5,
    SilentFOVColor = "White",

    AimbotWallCheck = false,
    AimbotVisibleCheck = false,
    AimbotShowFOV = true,
    AimbotFOVFill = true,
    AimbotFOVFillTransparency = 0.92,
    AimbotFOVAnimated = false,
    AimbotFOVColor = "White",
    AimbotSmooth = 0.15,

    WeatherEnabled = false,
    AmbientSoundEnabled = false,
    AmbientSoundType = "Rain",
    AmbientSoundVolume = 0.45,
    AmbientSoundPitch = 1,
    Weather = false,
    WeatherType = "Rain",
    WeatherIntensity = 1,
    WeatherSoundVolume = 0.6,
    WeatherStorm = false,
    WeatherStormMin = 8,
    WeatherStormVar = 6,
    WeatherMeteors = false,
    WeatherMeteorRate = 1,
    WeatherShootingStars = false,
    WeatherStarRate = 1,
    WeatherPuddles = false,
    WeatherMood = true,
    WeatherClockDial = false,
    WeatherRainbow = false,
    WeatherGodRays = false,
    SkyboxPreset = "Off",

    EmoteEnabled = false,
    EmoteId = "5917459365",
    EmoteName = "Floss",
    EmoteSpeed = 1,
    EmoteLoop = true,
    EmoteKeepPlaying = true,
    EmoteAlwaysOn = true,
    EmotePriority = "Action4",

    AutoClickEnabled = false,
    AutoClickCPS = 16,
    AutoClickOnlyOnTarget = true,

    NoRecoil = false,
    NoRecoilStrength = 1,
    NoRecoilOnlyShoot = true,
    NoRecoilScanGun = false,

    RapidFire = false,
    RapidFireValue = 999,
    RapidFireDelay = 0.001,
    RapidFireLoop = true,
    FastMelee = false,
    ItemLibraryKeepAlive = true,

    VoidSpamEnabled = false,
    VoidMethod = "Quantum",
    VoidSpeed = 100000000000,
    VoidChaos = 0.98,
    VoidBaseAltitude = 100000000000, -- distancia extrema
    VoidRadius = 100000000000,
    VoidEvade = true,
    VoidEvadeDist = 100000000000,
    VoidEvadeRadius = 100000000000,
    VoidEvadeSpeed = 100000000000,
    VoidEvadeCooldown = 0.05,

    PredDodgeEnabled = false,
    PredRadius = 80,
    PredDodgeDist = 120,
    PredCooldown = 0.3,
    PredThreshold = 800,
    PredMult = 1,

    TriggerbotEnabled = false,
    TriggerbotFOV = 120,
    TriggerbotCPS = 12,
    TriggerbotPart = "Head",

    AutoEquipMelee = false,
    AutoEquipEnabled = false,
    AutoEquipName = "Chainsaw",
    AutoEquipAny = false,
    AutoEquipSlot = 3, -- 1 primary 2 secondary 3 melee
    AutoEquipInterval = 0.35,
    AutoEquipDelay = 0.8,
    MeleeName = "Chainsaw",
    DesyncJitter = false,
    DesyncJitterPower = 12,

    -- HVH Defense
    HurtEvade = false,
    HurtEvadeDist = 55,
    HurtEvadeCooldown = 0.35,
    NearEnemyEvade = false,
    NearEnemyDist = 25,
    NearEnemyPush = 45,
    AdaptiveDesync = false,
    AdaptiveDesyncPower = 18,
    AdaptiveDesyncRange = 200,
    VelocityBreak = false,
    VelocityBreakMax = 140,
    RandomMicroMove = false,
    RandomMicroPower = 3,
    AntiMeleeRange = 20,
    AntiMeleePush = 40,
    AntiMeleeEnabled = false,
    LowHPPanic = false,
    LowHPThreshold = 30,
    LowHPBoostDist = 80,

    -- Anti UE / Kicia HVH
    InfiniteRange = true,          -- ignora MaxRange en target
    AntiUEClose = false,           -- si UE se pega cerca → push lejos
    AntiUECloseDist = 35,
    AntiUEClosePush = 80,
    AntiUEPeek = false,            -- side peek al detectar rager cerca
    AntiUEPeekDist = 40,
    RageInfinite = true,           -- rage sin límite de distancia
    StickFarMode = false,          -- stick más lejos (anti melee UE)
    StickFarMult = 2.5,
    DesyncSpam = false,            -- micro desync continuo anti hit
    DesyncSpamPower = 8,
    HitboxBig = false,
    HitboxBigSize = 18,


    AntiKick = true,
    SoftStick = true,
    SoftStickSpeed = 180,
    SoftStickMaxStep = 5,
    -- TP atras del enemigo (sin delay)
    StickBehindTP = false,
    StickBehindDist = 10,   -- offset atras del enemigo
    StickBehindHeight = 1.5,
    StickBehindRange = math.huge, -- alcance infinito para TP
    CameraIndependent = true,
    BlockIllegalNotify = true,

    CameraLookAtTarget = false,
    ForceZeroVelocity = true,
    StickOnRender = true,
    LockYWhileStick = false,

    Fullbright = false,
    Brightness = 2,
    Exposure = 0,
    ClockTime = 14,
    NoFog = true,
    Saturation = 0,
    Contrast = 0,
    BloomEnabled = false,
    BloomIntensity = 0.4,
    BlurEnabled = false,
    BlurSize = 8,
    AmbientColor = false,
    LightingOverride = false,
    AmbientR = 120, AmbientG = 120, AmbientB = 140,
    OutdoorR = 140, OutdoorG = 140, OutdoorB = 160,
    ColorShiftEnabled = false,
    ColorShiftR = 0, ColorShiftG = 0, ColorShiftB = 0,
    TintEnabled = false,
    TintR = 255, TintG = 255, TintB = 255,
    ShadowsEnabled = true,
    GlobalShadows = true,
    EnvironmentDiffuse = 1,
    EnvironmentSpecular = 1,
    WallXray = false,
    WallTransparency = 0.55,
    WallLocalOnly = true, -- solo client
    MapColorEnabled = false,
    MapColorR = 80, MapColorG = 80, MapColorB = 100,
    RemoveTextures = false,
    GlassWindows = false,
    CharacterTransparency = 0,
    ApplyCharVisualsEnabled = false,
    ForceFieldVisual = false,
    ForceFieldColor = Color3.fromRGB(180, 0, 255),
    ForceFieldTransparency = 0.72,
    ForceFieldOutlineTransparency = 0.55,
    ForceFieldStyle = "Material", -- Material (solo skin) | Highlight | Both | Soft
    ForceFieldPulse = false,
    ForceFieldRemoveAcc = false,
    RemoveAccessories = false,

    ESPEnabled = false,
    ESPPerfMode = true, -- OPT: ESP ligero high FPS
    ESPMaxPlayers = 16, -- mas jugadores
    ESPUpdateInterval = 0.033, -- ~30 Hz fluido
    ESPRadar = false,
    ESPChams = false,

    -- LuaHook ESP
    ESP = false,
    ESPBox = true,
    ESPBoxStyle = "Full Box",
    ESPBoxBrackets = false,
    ESPBoxFill = false,
    ESPBoxThickness = 1,
    ESPBoxColorMode = "Solid",
    ESPBoxColor = Color3.fromRGB(255, 255, 255),
    ESPBoxGradA = Color3.fromRGB(255, 59, 78),
    ESPBoxGradB = Color3.fromRGB(255, 194, 75),
    ESPBoxFillColor = Color3.fromRGB(255, 59, 78),
    ESPBoxTransparency = 0,
    ESPBoxFillTransparency = 0.75,
    ESPBoxScale = 1,
    ESPName = true,
    ESPDistance = true,
    ESPHealth = true,
    ESPHealthBar = true,
    ESPSkeleton = false, -- OPT: skeleton mata FPS

    ESPTracers = false,
    ESPTeamCheck = false,
    ESPMaxDistance = 100000, -- 0 = infinite
    ESPDistanceScaling = true,
    ESPDistanceScalingRef = 50,
    ESPChams = false,
    ESPFlags = false, -- OPT


    ESPMaxDist = 1e9, -- infinito
    ESPInfiniteRange = true,
    ESPTeamCheck = true,
    ESPShowName = true,
    ESPShowDist = true,
    ESPShowHP = true,
    ESPShowBox = true,
    ESPBoxStyle = "Corner", -- Corner / Full / Both
    ESPSkeleton = false,
    ESPGlow = true,
    ESPSkeletonThickness = 1.5,
    ESPTracers = false,
    ESPTracerFrom = "Bottom",
    ESPColor = "White",
    ESPHealthBarColor = "Green",
    ESPUseTeamColor = false,
    ESPUseHPColor = false,
    ESPBoxThickness = 2,
    ESPCasingThickness = 2,
    ESPFilledBox = false,
    ESPFilledBoxTrans = 0.85,

    -- Bullet tracers
    BulletTracerEnabled = false,
    BulletTracerColor = "Cyan",
    BulletTracerDuration = 2.5,
    BulletTracerThickness = 1.5,
    BulletTracerFromGun = true,
    BulletTracerStyle = "Line",
    BulletTracerGlow = 0.45,
    BulletTracerFade = 0.5,

    -- Stretched resolution
    StretchedRes = false,
    StretchedAspect = 1.33, -- legacy
    StretchedFOV = 90,
    StretchedWidth = 100,  -- % ancho (source style)
    StretchedHeight = 55, -- mas aplastado (0.55)
    StretchedUseMatrix = false, -- OFF default: matrix congela; FOV solo es estable

    ThirdPerson = false,
    ThirdPersonDistance = 10,
    ThirdPersonHeight = 2.5,
    ThirdPersonSmooth = 0.2,
    Watermark = false,
    Crosshair = false,
    CrosshairSize = 25,
    CrosshairGap = 10,
    CrosshairThickness = 2,
    CrosshairOutline = true,
    CrosshairSpin = true,
    CrosshairSpinSpeed = 120,
    CrosshairText = "",
    CrosshairTextSize = 22,
    CrosshairRainbow = true,
    CrosshairOffsetX = 0,
    CrosshairOffsetY = 0,
    CrosshairTextOffsetY = 28,
    CurrentSky = "None",
    AutoRespawn = false,

    -- Spoofer
    SpoofEnabled = false,
    SpoofMyName = false,
    SpoofMyNameValue = "shako.win",
    SpoofMyUserValue = "",     -- @username falso en perfil/UI (si vacio usa MyName)
    SpoofOthers = false,
    SpoofOthersValue = "User",
    SpoofTargetName = "",
    SpoofTargetFake = "",
    SpoofHideReal = true,
    SpoofRandomOthers = false,
    SpoofProfileUI = true,     -- reescribir perfil / PlayerGui
    -- Device spoofer
    DeviceSpoofEnabled = false,
    DeviceSpoofType = "Mobile", -- PC, Mobile, Console, VR, Controller
    DeviceSpoofAttribute = true, -- setea atributos locales por si el juego los lee

    -- Skinchanger
    SkinchangerEnabled = false,
    SkinTargetUserId = 1,
    ShakoSyncEnabled = true, -- ver avatars de otros shako.win

    SkinTargetName = "",
    SkinOnRespawn = true,

    -- Keybinds (Enum.KeyCode name)
    KeybindStick = "V",
    KeybindRage = "R",
    KeybindSilent = "T",
    KeybindAimbot = "Y",
    KeybindFly = "F",
    KeybindNoclip = "G",
    KeybindAntiAim = "B",
    KeybindStrafe = "C",
    KeybindOrbit = "X",
    KeybindSpeed = "Z",
    KeybindVoidSpam = "H",
}


local config = getgenv().config

-- ==================== RIVALS MODULES + ITEMLIBRARY ====================
local RS = game:GetService("ReplicatedStorage")
local util, enum, FighterController, SpectateController
local modulesOK = false
local ItemBackup = {}
local gunExceptions = { RPG = true } -- sniper/bow/crossbow SI tienen rapid + no spread
local rageLastFire = 0
local deflecting = {}
local rageSlots = { Primary = 1, Secondary = 2, Melee = 3 }

function loadRivalsModules()
    local ok1, u = pcall(function() return require(RS.Modules.Utility) end)
    local ok2, e = pcall(function() return require(RS.Modules.EnumLibrary) end)
    local ok3, fc = pcall(function()
        return require(LocalPlayer.PlayerScripts.Controllers.FighterController)
    end)
    local ok4, sc = pcall(function()
        return require(LocalPlayer.PlayerScripts.Controllers:WaitForChild("SpectateController"))
    end)
    if ok1 and ok2 and ok3 then
        util, enum, FighterController = u, e, fc
        if ok4 then SpectateController = sc end
        modulesOK = true
        return true
    end
    modulesOK = false
    return false
end

task.spawn(function()
    for _ = 1, 40 do
        if loadRivalsModules() then
            pcall(function() Notify("Rivals", "FighterController OK") end)
            break
        end
        task.wait(0.4)
    end
end)

function getItems()
    local ok, mod = pcall(function() return require(RS.Modules.ItemLibrary) end)
    if ok and mod and mod.Items then return mod.Items end
    return nil
end

function ilSave(name, data, prop)
    if data[prop] == nil then return end
    ItemBackup[name] = ItemBackup[name] or {}
    if ItemBackup[name][prop] == nil then ItemBackup[name][prop] = data[prop] end
end

function ilSet(name, data, prop, val)
    if data[prop] == nil then return end
    ilSave(name, data, prop)
    data[prop] = val
end

function ilRestore(name, data)
    local b = ItemBackup[name]
    if not b then return end
    for prop, old in pairs(b) do
        if data[prop] ~= nil then data[prop] = old end
    end
end

function ApplyItemLibraryGuns(on)
    local Items = getItems()
    if not Items then return false end
    for name, data in pairs(Items) do
        if typeof(data) ~= "table" then continue end
        local lname = string.lower(tostring(name))
        -- solo excluir RPG / explosivos
        if gunExceptions[name] or lname:find("rpg", 1, true) or lname:find("rocket", 1, true) then
            continue
        end
        if on then
            -- No spread / no recoil (incluye sniper)
            if config.NoRecoil or config.RapidFire then
                ilSet(name, data, "ShootSpread", 0)
                ilSet(name, data, "ShootRecoil", 0)
                ilSet(name, data, "Recoil", 0)
                ilSet(name, data, "Spread", 0)
                ilSet(name, data, "MinSpread", 0)
                ilSet(name, data, "MaxSpread", 0)
                ilSet(name, data, "AimSpread", 0)
                ilSet(name, data, "HipSpread", 0)
            end
            ilSet(name, data, "ShootAccuracy", 0)
            ilSet(name, data, "Accuracy", 0)
            -- Rapid fire (incluye sniper)
            local delay = config.RapidFireDelay or 0.001
            ilSet(name, data, "ShootCooldown", delay)
            ilSet(name, data, "ShootBurstCooldown", delay)
            ilSet(name, data, "FireRate", delay)
            ilSet(name, data, "Cooldown", delay)
            ilSet(name, data, "ReloadTime", data.ReloadTime) -- no tocar reload
            -- sniper charge / bolt
            ilSet(name, data, "ChargeTime", 0)
            ilSet(name, data, "BoltTime", 0.001)
            ilSet(name, data, "PumpTime", 0.001)
            ilSet(name, data, "CycleTime", 0.001)
        else
            ilRestore(name, data)
        end
    end
    return true
end

function ApplyItemLibraryMelee(on)
    local Items = getItems()
    if not Items then return false end
    for name, data in pairs(Items) do
        if typeof(data) ~= "table" then continue end
        if on then
            ilSet(name, data, "AttackCooldown", 0.001)
            ilSet(name, data, "SwingCooldown", 0.001)
            ilSet(name, data, "MeleeCooldown", 0.001)
            ilSet(name, data, "Cooldown", 0.001)
            ilSet(name, data, "RecoveryTime", 0.001)
            ilSet(name, data, "ResetTime", 0.001)
        else
            ilRestore(name, data)
        end
    end
    return true
end

task.spawn(function()
    while true do
        task.wait(5)
        if config.ItemLibraryKeepAlive then
            if config.RapidFire then pcall(ApplyItemLibraryGuns, true) end
            if config.FastMelee then pcall(ApplyItemLibraryMelee, true) end
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(4) -- OPT
        if not config.RagebotEnabled or not modulesOK then continue end
        local lf = FighterController and FighterController.LocalFighter
        if lf then
            local slot = rageSlots[config.RageWeaponSlot or "Melee"] or 3
            pcall(function() lf:EquipItem(slot) end)
        end
    end
end)

Players.PlayerRemoving:Connect(function(p) deflecting[p] = nil end)

function updateDeflection()
    if not FighterController or not FighterController.Objects then return end
    for _, fighterObj in pairs(FighterController.Objects) do
        local player = fighterObj.Player
        if not player then continue end
        if not fighterObj.Entity or not fighterObj.Entity:IsAlive() or fighterObj:Get("IsSpectating") then
            deflecting[player] = false
            continue
        end
        local equipped = fighterObj.EquippedItem
        local isKatana = equipped and equipped.ViewModel and equipped.ViewModel.Name == "Katana"
        local isDeflecting = false
        if isKatana then
            isDeflecting = (equipped._attack_cooldown and equipped._attack_cooldown > tick()) or false
        end
        deflecting[player] = isDeflecting
    end
end

-- FFA: si casi todos comparten el mismo TeamID → Free For All
function isFFAMode()
    if config.RagebotForceFFA then return true end
    local teams, alive = {}, 0
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if not plr.Character then continue end
        local hum = plr.Character:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health <= 0 then continue end
        alive = alive + 1
        local tid = getPlayerTeamId and getPlayerTeamId(plr) or nil
        -- nil / 0 / "0" / "FFA" cuentan como FFA-ish
        local key = (tid == nil or tid == "0" or tid == "nil" or tid == "FFA") and "__ffa" or tostring(tid)
        teams[key] = true
    end
    if alive <= 1 then return true end
    local unique = 0
    for _ in pairs(teams) do unique = unique + 1 end
    -- un solo "equipo" en el server = Free For All
    if unique <= 1 then return true end
    return false
end

function isEnemyRageRivals(player)
    if not player or player == LocalPlayer then return false end
    if not player.Character then return false end
    -- solo targets en pelea real (FighterController), no lobby/spectate
    if FighterController and FighterController.Objects then
        local foundAlive = false
        for _, fo in pairs(FighterController.Objects) do
            if fo and fo.Player == player then
                if fo.Entity and fo.Entity:IsAlive() and not fo:Get("IsSpectating") then
                    foundAlive = true
                end
                break
            end
        end
        if not foundAlive then return false end
    end
    -- Team check OFF o FFA forzado = atacar a todos
    if config.RagebotTeamCheck == false then return true end
    if config.RagebotForceFFA == true then return true end
    -- auto FFA (Rivals Free For All suele poner el mismo TeamID a todos)
    if isFFAMode() then return true end
    -- ranked / teams: solo enemigos reales
    -- NO usar config.TeamCheck global (rompe rage en FFA)
    if IsSameTeam and IsSameTeam(player) then return false end
    return true
end

function hasKnifeViewModel(targetPlayer)
    if not targetPlayer then return false end
    local viewModels = workspace:FindFirstChild("ViewModels")
    if not viewModels then return false end
    local targetName = targetPlayer.Name
    for _, model in ipairs(viewModels:GetChildren()) do
        if model:IsA("Model")
            and string.find(model.Name, targetName, 1, true)
            and string.find(model.Name, "Knife", 1, true) then
            return true
        end
    end
    return false
end

function fireRageUseItem(item, desyncCF, targetHead, targetRoot)
    if not util or not enum or not item then return end
    local originPos = desyncCF and desyncCF.Position or targetRoot.Position
    local targetPos = targetHead.Position
    local aimCF = CFrame.lookAt(originPos, targetPos)
    local targetCF = targetHead.CFrame
    local randomOffset = Vector3.new(
        (math.random() - 0.5) * 0.1,
        (math.random() - 0.5) * 0.1,
        (math.random() - 0.5) * 0.1
    )
    local aimedPos = targetPos + randomOffset
    local objSpaceHeadOffset = targetHead.CFrame:ToObjectSpace(CFrame.new(aimedPos))
    local cameradata = {}
    cameradata[utf8.char(1)] = {
        [utf8.char(0)] = util:EncodeCFrame(aimCF),
        [utf8.char(1)] = util:EncodeCFrame(targetCF),
        [utf8.char(2)] = targetHead,
        [utf8.char(3)] = util:EncodeCFrame(objSpaceHeadOffset)
    }
    pcall(function()
        RS.Remotes.Replication.Fighter.UseItem:FireServer(
            item:Get("ObjectID"),
            enum:ToEnum("StartShooting"),
            cameradata,
            nil
        )
    end)
end

-- ==================== SLEEPY COUNTER + KICIA COUNTER ====================
local sleepyLastFire, kiciaLastFire = 0, 0
local kiciaOrbitAng = 0

local function counterPickTarget(teamCheck, maxRange)
    local my = getHRP and getHRP() or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart"))
    if not my then return nil, nil, nil end
    local best, bestD, bestHead, bestRoot = nil, maxRange or 1e12, nil, nil
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if teamCheck and IsEnemy and not IsEnemy(plr) then continue end
        if teamCheck and IsSameTeam and IsSameTeam(plr) then continue end
        local char = plr.Character
        if not char then continue end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local head = char:FindFirstChild("Head")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not head or not root or hum.Health <= 0 then continue end
        if char:FindFirstChildOfClass("ForceField") then continue end
        local d = (my.Position - root.Position).Magnitude
        if d < bestD then
            bestD, best, bestHead, bestRoot = d, plr, head, root
        end
    end
    return best, bestHead, bestRoot
end

local function getHandSideCFrame(root, head, sideDist, handY, height)
    -- lado de la mano derecha del enemigo, LEJOS (no pegado)
    local rf = root.CFrame
    local right = rf.RightVector
    local dist = tonumber(sideDist) or 14
    local yOff = tonumber(handY) or 0.5
    local h = tonumber(height) or 1.2
    -- posición al lado de la mano (right * dist), ligeramente adelante
    local handPos = root.Position + right * dist + rf.LookVector * 2 + Vector3.new(0, h + yOff, 0)
    local look = head and head.Position or (root.Position + Vector3.new(0, 1.5, 0))
    return CFrame.lookAt(handPos, look), handPos
end

function StartSleepyCounter()
    SafeDisconnect("SleepyCounter")
    config.SleepyCounterEnabled = true
    Connections.SleepyCounter = RunService.Heartbeat:Connect(function()
        if not config.SleepyCounterEnabled then return end
        if not CanDoDangerousMove or not CanDoDangerousMove() then return end
        if IsAntiUnnamedActive and IsAntiUnnamedActive() then return end
        -- no pelear con rage normal / void al mismo tiempo si UE counter
        local my = getHRP and getHRP()
        if not my then return end
        local teamCheck = config.SleepyCounterTeamCheck ~= false
        local plr, head, root = counterPickTarget(teamCheck, config.SleepyCounterRange or 1e12)
        if not plr or not head or not root then return end

        local cf, pos = getHandSideCFrame(
            root, head,
            config.SleepyCounterSideDist or 14,
            config.SleepyCounterHandY or 0.5,
            config.SleepyCounterHeight or 1.2
        )
        -- body al lado de la mano (lejitos)
        pcall(function()
            my.CFrame = cf
            my.AssemblyLinearVelocity = Vector3.zero
        end)

        -- auto fire cabeza
        local rate = tonumber(config.SleepyCounterFireRate) or 0.02
        if tick() - sleepyLastFire < rate then return end
        sleepyLastFire = tick()
        local lf = FighterController and FighterController.LocalFighter
        local item = lf and lf.EquippedItem
        if item and fireRageUseItem then
            fireRageUseItem(item, cf, head, root)
        end
    end)
    Notify("Sleepy Counter", "ON · lado mano + head")
end

function StopSleepyCounter()
    config.SleepyCounterEnabled = false
    SafeDisconnect("SleepyCounter")
    Notify("Sleepy Counter", "OFF")
end

-- Kicia Counter: TP al CIELO, se mueve de lado a lado (linea), al OFF vuelve
local kiciaSavedCF = nil
local kiciaLineT = 0 -- 0..1 ping-pong
local kiciaLineDir = 1
local kiciaSkyOrigin = nil -- centro de la linea en el cielo

function StartKiciaCounter()
    SafeDisconnect("KiciaCounter")
    config.KiciaCounterEnabled = true
    kiciaLineT = 0.5
    kiciaLineDir = 1
    kiciaSavedCF = nil
    kiciaSkyOrigin = nil

    local my0 = getHRP and getHRP()
    if my0 then
        kiciaSavedCF = my0.CFrame
        local skyY = tonumber(config.KiciaCounterSkyY) or 450
        kiciaSkyOrigin = Vector3.new(my0.Position.X, my0.Position.Y + skyY, my0.Position.Z)
        -- TP inicial al cielo (centro)
        pcall(function()
            my0.CFrame = CFrame.new(kiciaSkyOrigin)
            my0.AssemblyLinearVelocity = Vector3.zero
        end)
    end

    Connections.KiciaCounter = RunService.Heartbeat:Connect(function(dt)
        if not config.KiciaCounterEnabled then return end
        local my = getHRP and getHRP()
        if not my then return end

        -- si no hay origin (respawn), recrear desde pos actual
        if not kiciaSkyOrigin then
            if not kiciaSavedCF then kiciaSavedCF = my.CFrame end
            local skyY = tonumber(config.KiciaCounterSkyY) or 450
            kiciaSkyOrigin = Vector3.new(my.Position.X, (kiciaSavedCF and kiciaSavedCF.Position.Y or my.Position.Y) + skyY, my.Position.Z)
        end

        local width = tonumber(config.KiciaCounterLineWidth) or 80
        local speed = tonumber(config.KiciaCounterMoveSpeed) or 400
        -- avance en la linea (ping-pong 0..1)
        local half = width * 0.5
        -- studs por segundo / ancho total → delta de t
        local span = math.max(width, 1)
        kiciaLineT = kiciaLineT + (kiciaLineDir * (speed / span) * (dt or 0.016))
        if kiciaLineT >= 1 then
            kiciaLineT = 1
            kiciaLineDir = -1
        elseif kiciaLineT <= 0 then
            kiciaLineT = 0
            kiciaLineDir = 1
        end
        -- linea en eje X local del cielo (lado a lado)
        local xOff = (kiciaLineT * 2 - 1) * half -- -half .. +half
        local pos = kiciaSkyOrigin + Vector3.new(xOff, 0, 0)

        -- mirar al target si hay (para disparar), si no mirar abajo
        local look = pos + Vector3.new(0, -10, 0)
        local teamCheck = config.KiciaCounterTeamCheck ~= false
        local plr, head, root = counterPickTarget(teamCheck, 1e12)
        if head then look = head.Position end

        local cf = CFrame.lookAt(pos, look)
        pcall(function()
            my.CFrame = cf
            my.AssemblyLinearVelocity = Vector3.zero
            my.AssemblyAngularVelocity = Vector3.zero
        end)

        -- opcional: seguir pegando a la cabeza desde el cielo
        if config.KiciaCounterShoot ~= false and head and root then
            local rate = tonumber(config.KiciaCounterFireRate) or 0.015
            if tick() - kiciaLastFire >= rate then
                kiciaLastFire = tick()
                local lf = FighterController and FighterController.LocalFighter
                local item = lf and lf.EquippedItem
                if item and fireRageUseItem then
                    fireRageUseItem(item, cf, head, root)
                end
            end
        end
    end)
    Notify("Kicia Counter", "ON · cielo + linea")
end

function StopKiciaCounter()
    config.KiciaCounterEnabled = false
    SafeDisconnect("KiciaCounter")
    -- volver a donde estabas
    local my = getHRP and getHRP()
    if my and kiciaSavedCF then
        pcall(function()
            my.CFrame = kiciaSavedCF
            my.AssemblyLinearVelocity = Vector3.zero
        end)
        Notify("Kicia Counter", "OFF · regresaste")
    else
        Notify("Kicia Counter", "OFF")
    end
    kiciaSkyOrigin = nil
    -- no borrar saved hasta el next start
end




local Connections = {
    Main = nil, StickRender = nil, Fly = nil, AntiUnnamed = nil,
    Noclip = nil, AntiAim = nil, Ragebot = nil, AutoClick = nil,
    NoRecoil = nil, AUWatch = nil, ESP = nil, VisualLoop = nil,
    Aimbot = nil, AimbotInput = nil, AimbotInputEnd = nil, RapidFireChar = nil, VoidSpam = nil,
    PredDodge = nil, Triggerbot = nil, SilentFOV = nil, FOVDraw = nil, Weather = nil, Defense = nil,
    SleepyCounter = nil, KiciaCounter = nil, StickBehind = nil
}

local CurrentTarget, FakePart, RagebotTarget, LockedTarget
local OrbitAngle, SpinAngle, JitterSide = 0, 0, 1
local stickPosIndex, stickPosLast = 1, 0
local CanMove, SpawnTime = false, 0
local LastClick, lastCamCF = 0, nil
local UECounterReady = false
local CoordsSpinAngle = 0
local LastRageSwitch, lastRecoilScan = 0, 0
local lastPredDodge, lastTriggerClick, lastVoidRage = 0, 0, 0
local voidRagePhase = 0
local selectedConfig, configNameInput = "(ninguna)", "default"
local ESPFolder, CrossDraws, WatermarkDraw = {}, {}, nil
-- Drawing API (algunos executors)
pcall(function()
    if not Drawing and getgenv and getgenv().Drawing then Drawing = getgenv().Drawing end
end)

local RapidFireSaved, RapidFireLoopRunning, RecoilSaved = {}, false, {}
local voidElapsed, voidPos, voidDriftDir, voidBase, voidEvadeCD, intendedVoidPos =
    0, Vector3.new(0, 100000000000, 0), Vector3.new(1, 0, 0), Vector3.new(0, 100000000000, 0), 0, Vector3.new(0, 100000000000, 0)

-- Config system (tabla para ahorrar registers Luau)
local Cfg = {
    folder = "shako_configs",
    autoload = "shako_configs/_autoload.txt",
}

-- watermark removed
pcall(function()
    for i = 1, 12 do
        local l = Drawing.new("Line")
        l.Thickness = 1.5
        l.Color = Color3.fromRGB(255, 255, 255)
        l.Visible = false
        CrossDraws[i] = l
    end
    CrossDraws.Dot = Drawing.new("Circle")
    CrossDraws.Dot.Filled = true
    CrossDraws.Dot.Thickness = 0
    CrossDraws.Dot.Radius = 2
    CrossDraws.Dot.Visible = false
    CrossDraws.DotOutline = Drawing.new("Circle")
    CrossDraws.DotOutline.Filled = false
    CrossDraws.DotOutline.Thickness = 1
    CrossDraws.DotOutline.Radius = 3
    CrossDraws.DotOutline.Visible = false
end)

local CC = Lighting:FindFirstChild("SleepyCC") or Instance.new("ColorCorrectionEffect")
CC.Name = "SleepyCC"
CC.Parent = Lighting
local BloomFX = Lighting:FindFirstChild("SleepyBloom") or Instance.new("BloomEffect")
BloomFX.Name = "SleepyBloom"
BloomFX.Enabled = false
BloomFX.Parent = Lighting
local BlurFX = Lighting:FindFirstChild("SleepyBlur") or Instance.new("BlurEffect")
BlurFX.Name = "SleepyBlur"
BlurFX.Enabled = false
BlurFX.Parent = Lighting

local oldLighting = {
    Brightness = Lighting.Brightness,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
    ExposureCompensation = Lighting.ExposureCompensation,
    ClockTime = Lighting.ClockTime,
    FogStart = Lighting.FogStart,
    FogEnd = Lighting.FogEnd
}

local Skies = {
    ["Dark Sky"] = {
        SkyboxUp = "rbxassetid://570555929", SkyboxRt = "rbxassetid://570555882",
        SkyboxDn = "rbxassetid://570555964", SkyboxFt = "rbxassetid://570555800",
        SkyboxLf = "rbxassetid://570555840", SkyboxBk = "rbxassetid://570555736"
    },
    ["Nebula"] = {
        SkyboxUp = "rbxassetid://159454288", SkyboxRt = "rbxassetid://159454300",
        SkyboxLf = "rbxassetid://159454286", SkyboxFt = "rbxassetid://159454293",
        SkyboxBk = "rbxassetid://159454299", SkyboxDn = "rbxassetid://159454296"
    },
    ["Clouds"] = {
        SkyboxUp = "rbxassetid://570557727", SkyboxRt = "rbxassetid://570557672",
        SkyboxLf = "rbxassetid://570557620", SkyboxFt = "rbxassetid://570557559",
        SkyboxBk = "rbxassetid://570557514", SkyboxDn = "rbxassetid://570557775"
    },
    ["Vaporwave"] = {
        SkyboxUp = "rbxassetid://1417494643", SkyboxRt = "rbxassetid://1417494499",
        SkyboxLf = "rbxassetid://1417494402", SkyboxFt = "rbxassetid://1417494253",
        SkyboxBk = "rbxassetid://1417494030", SkyboxDn = "rbxassetid://1417494146"
    },
    ["Twilight"] = {
        SkyboxUp = "rbxassetid://264907379", SkyboxRt = "rbxassetid://264908886",
        SkyboxLf = "rbxassetid://264909758", SkyboxFt = "rbxassetid://264909420",
        SkyboxBk = "rbxassetid://264908339", SkyboxDn = "rbxassetid://264907909"
    },
    ["Art Mountain"] = {
        SkyboxUp = "rbxassetid://2128462236", SkyboxRt = "rbxassetid://2128462027",
        SkyboxLf = "rbxassetid://2128462027", SkyboxFt = "rbxassetid://2128458653",
        SkyboxBk = "rbxassetid://2128458653", SkyboxDn = "rbxassetid://2128462480"
    },
    ["Alien Red"] = {
        SkyboxUp = "rbxassetid://7123412196", SkyboxRt = "rbxassetid://7123402505",
        SkyboxDn = "rbxassetid://7123387679", SkyboxFt = "rbxassetid://7123390433",
        SkyboxLf = "rbxassetid://7123394786", SkyboxBk = "rbxassetid://7123385217"
    }
}

function SetupAntiKick()
    if not config.AntiKick then return end
    pcall(function()
        local mt = getrawmetatable and getrawmetatable(game)
        if mt then
            setreadonly(mt, false)
            local old = mt.__namecall
            mt.__namecall = newcclosure(function(self, ...)
                local method = tostring(getnamecallmethod and getnamecallmethod() or "")
                if method == "Kick" or method == "kick" then
                    if config.BlockIllegalNotify then
                        pcall(function() Notify("AntiKick", "Kick bloqueado") end)
                    end
                    return nil
                end
                return old(self, ...)
            end)
            setreadonly(mt, true)
        end
    end)
    pcall(function()
        if hookfunction and typeof(LocalPlayer.Kick) == "function" then
            hookfunction(LocalPlayer.Kick, newcclosure(function()
                if config.BlockIllegalNotify then
                    pcall(function() Notify("AntiKick", "Kick bloqueado") end)
                end
                return nil
            end))
        end
    end)
    pcall(function()
        LocalPlayer.Idled:Connect(function()
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        end)
    end)
end

function SafeDisconnect(name)
    if Connections[name] then
        pcall(function() Connections[name]:Disconnect() end)
        Connections[name] = nil
    end
end
function getHRP()
    return LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
end
function getHum()
    return LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
end
function getPlayerTeamId(plr)
    if not plr then return nil end
    local tid = nil
    pcall(function() tid = plr:GetAttribute("TeamID") end)
    if tid == nil then
        pcall(function() tid = plr:GetAttribute("Team") end)
    end
    if tid == nil and plr.Character then
        pcall(function() tid = plr.Character:GetAttribute("TeamID") end)
        if tid == nil then
            pcall(function() tid = plr.Character:GetAttribute("Team") end)
        end
    end
    -- FighterController object team
    if tid == nil and FighterController and FighterController.Objects then
        pcall(function()
            for _, obj in pairs(FighterController.Objects) do
                if obj.Player == plr then
                    tid = obj:Get("TeamID") or (obj.Get and obj:Get("Team"))
                    break
                end
            end
        end)
    end
    -- Spectate / Duel
    if tid == nil and SpectateController then
        pcall(function()
            local duel = SpectateController.CurrentDuelSubject
            if not duel or not duel.GetDueler then return end
            local dueler = duel:GetDueler(plr)
            if dueler then
                tid = dueler:Get("TeamID")
            end
        end)
    end
    if tid == nil and plr.Team then
        tid = plr.Team.Name
    end
    if tid == nil and plr.TeamColor then
        tid = tostring(plr.TeamColor)
    end
    if tid == nil then return nil end
    return tostring(tid)
end


function IsRivalsBot(plr)
    if not plr then return true end
    -- attributes comunes Rivals / bots
    local ok, a = pcall(function()
        return plr:GetAttribute("IsBot")
            or plr:GetAttribute("Bot")
            or plr:GetAttribute("IsNPC")
            or plr:GetAttribute("NPC")
            or plr:GetAttribute("IsTrainingBot")
            or plr:GetAttribute("TrainingBot")
    end)
    if ok and a then return true end
    local name = string.lower(tostring(plr.Name or ""))
    local disp = string.lower(tostring(plr.DisplayName or ""))
    if name:find("bot", 1, true) or disp:find("bot", 1, true) then return true end
    if name:find("npc", 1, true) or disp:find("npc", 1, true) then return true end
    if name:find("dummy", 1, true) or disp:find("dummy", 1, true) then return true end
    if name:find("target", 1, true) and (name:find("train") or disp:find("train")) then return true end
    -- character attributes
    local char = plr.Character
    if char then
        local ok2, ca = pcall(function()
            return char:GetAttribute("IsBot")
                or char:GetAttribute("Bot")
                or char:GetAttribute("IsNPC")
                or char:GetAttribute("NPC")
        end)
        if ok2 and ca then return true end
    end
    -- players muy "fake" (a veces bots sin UserId real en studio)
    pcall(function()
        if plr.UserId and plr.UserId <= 0 then return true end
    end)
    if typeof(plr.UserId) == "number" and plr.UserId <= 0 then return true end
    return false
end

function IsSameTeam(plr)
    if not plr or plr == LocalPlayer then return true end -- self = no target

    local lTeam = getPlayerTeamId(LocalPlayer)
    local pTeam = getPlayerTeamId(plr)

    -- TeamID 0 / vacio = FFA, no son teammates
    local function isFFAId(t)
        if t == nil then return true end
        local s = tostring(t)
        return s == "0" or s == "" or s == "nil" or s == "FFA" or s == "None"
    end
    if isFFAId(lTeam) or isFFAId(pTeam) then
        return false
    end

    if lTeam ~= nil and pTeam ~= nil then
        return lTeam == pTeam
    end

    -- Roblox Team object
    local a, b = LocalPlayer.Team, plr.Team
    if a ~= nil and b ~= nil then
        return a == b
    end

    -- TeamColor (ignorar White default)
    local ac, bc = LocalPlayer.TeamColor, plr.TeamColor
    if ac and bc and tostring(ac) ~= "White" and tostring(bc) ~= "White" then
        return ac == bc
    end

    -- sin info de team: en modes FFA no son teammates
    return false
end

-- true = se puede targetear (enemigo real)
function IsEnemy(plr)
    if not plr or plr == LocalPlayer then return false end
    if not plr.Character then return false end
    -- TeamCheck OFF = todos enemigos (excepto yo)
    if config.TeamCheck == false then return true end
    -- TeamCheck ON = solo si NO mismo team
    return not IsSameTeam(plr)
end
function CanDoDangerousMove()
    -- si ya pasó delay o CanMove true
    if CanMove then return true end
    local delay = tonumber(config.SpawnDelay) or 0
    if delay <= 0 then return true end
    return (tick() - (SpawnTime or 0)) >= delay
end
function IsAntiUnnamedActive()
    -- alias: UE Counter activo
    return (config.UECounterEnabled or config.AntiUnnamed) == true and UECounterReady == true
end
-- Cámara: el script NO la toca (ni FOV, ni CFrame, ni Subject, ni tipo).
function LookAtPos(pos) end
function ResetCameraNormal() end
function EnsureCameraFree() end


function nameKey(n)
    return string.lower(tostring(n or "")):gsub("[%s_%-]", "")
end

function ApplySky(name)
    for _, v in pairs(Lighting:GetChildren()) do
        if v:IsA("Sky") then v:Destroy() end
    end
    if name == "None" or not Skies[name] then config.CurrentSky = "None" return end
    local sky = Instance.new("Sky")
    sky.Name = "CustomSky"
    for k, val in pairs(Skies[name]) do sky[k] = val end
    sky.Parent = Lighting
    config.CurrentSky = name
end
local _wallCache = _wallCache or {}
local _mapColorCache = _mapColorCache or {}

local function _mapRoots()
    local roots = {}
    for _, name in ipairs({"Map", "World", "Level", "Arena", "Stages", "Maps", "Geometry", "MapModel"}) do
        local f = workspace:FindFirstChild(name)
        if f then roots[#roots+1] = f end
    end
    if #roots == 0 then
        for _, c in ipairs(workspace:GetChildren()) do
            if (c:IsA("Model") or c:IsA("Folder")) and c.Name ~= "Players" then
                roots[#roots+1] = c
            end
        end
    end
    return roots
end

local function applyWallXray(on)
    local trans = tonumber(config.WallTransparency) or 0.55
    if on then
        local count = 0
        for _, root in ipairs(_mapRoots()) do
            for _, obj in ipairs(root:GetDescendants()) do
                if count > 2500 then break end -- OPT hard cap
                if obj:IsA("BasePart") and obj.CanCollide and obj.Transparency < 0.95 then
                    if LocalPlayer.Character and obj:IsDescendantOf(LocalPlayer.Character) then continue end
                    if not _wallCache[obj] then
                        _wallCache[obj] = { t = obj.Transparency }
                    end
                    pcall(function()
                        obj.LocalTransparencyModifier = math.clamp(trans, 0, 0.95)
                    end)
                    count += 1
                end
            end
        end
    else
        for obj, data in pairs(_wallCache) do
            if obj and obj.Parent then
                pcall(function()
                    obj.LocalTransparencyModifier = 0
                    if data.t ~= nil then obj.Transparency = data.t end
                end)
            end
        end
        table.clear(_wallCache)
    end
end

local function applyMapColor(on)
    if not on then
        for obj, col in pairs(_mapColorCache) do
            if obj and obj.Parent then pcall(function() obj.Color = col end) end
        end
        table.clear(_mapColorCache)
        return
    end
    local c = Color3.fromRGB(
        tonumber(config.MapColorR) or 80,
        tonumber(config.MapColorG) or 80,
        tonumber(config.MapColorB) or 100
    )
    local count = 0
    for _, root in ipairs(_mapRoots()) do
        for _, obj in ipairs(root:GetDescendants()) do
            if count > 2000 then break end
            if obj:IsA("BasePart") then
                if not _mapColorCache[obj] then _mapColorCache[obj] = obj.Color end
                pcall(function() obj.Color = c end)
                count += 1
            end
        end
    end
end

local function applyRemoveTextures(on)
    if not on then return end
    local count = 0
    for _, root in ipairs(_mapRoots()) do
        for _, obj in ipairs(root:GetDescendants()) do
            if count > 1500 then return end
            if obj:IsA("Texture") or obj:IsA("Decal") then
                pcall(function() obj.Transparency = 1 end)
                count += 1
            end
        end
    end
end

function ApplyVisuals()
    pcall(function()
        if config.Fullbright then
            Lighting.Brightness = 3
            Lighting.Ambient = Color3.new(1, 1, 1)
            Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
            Lighting.ClockTime = 12
            Lighting.ExposureCompensation = 0.6
            Lighting.GlobalShadows = false
        elseif config.LightingOverride then
            Lighting.Brightness = tonumber(config.Brightness) or 2
            Lighting.ExposureCompensation = tonumber(config.Exposure) or 0
            Lighting.ClockTime = tonumber(config.ClockTime) or 14
            Lighting.Ambient = Color3.fromRGB(
                tonumber(config.AmbientR) or 120,
                tonumber(config.AmbientG) or 120,
                tonumber(config.AmbientB) or 140
            )
            Lighting.OutdoorAmbient = Color3.fromRGB(
                tonumber(config.OutdoorR) or 140,
                tonumber(config.OutdoorG) or 140,
                tonumber(config.OutdoorB) or 160
            )
            if config.GlobalShadows ~= nil then
                Lighting.GlobalShadows = config.GlobalShadows and true or false
            end
            pcall(function()
                Lighting.EnvironmentDiffuseScale = tonumber(config.EnvironmentDiffuse) or 1
                Lighting.EnvironmentSpecularScale = tonumber(config.EnvironmentSpecular) or 1
            end)
        else
            if not config.Fullbright then
                Lighting.Brightness = tonumber(config.Brightness) or oldLighting.Brightness
                Lighting.ExposureCompensation = tonumber(config.Exposure) or oldLighting.ExposureCompensation
                Lighting.ClockTime = tonumber(config.ClockTime) or oldLighting.ClockTime
            end
            if config.AmbientColor then
                Lighting.Ambient = Color3.fromRGB(
                    tonumber(config.AmbientR) or 120,
                    tonumber(config.AmbientG) or 120,
                    tonumber(config.AmbientB) or 140
                )
                Lighting.OutdoorAmbient = Color3.fromRGB(
                    tonumber(config.OutdoorR) or 140,
                    tonumber(config.OutdoorG) or 140,
                    tonumber(config.OutdoorB) or 160
                )
            end
        end

        if config.NoFog then
            Lighting.FogStart = 1e9
            Lighting.FogEnd = 1e9
            pcall(function() Lighting.Atmosphere.Density = 0 end)
        else
            Lighting.FogStart = oldLighting.FogStart
            Lighting.FogEnd = oldLighting.FogEnd
        end

        -- ColorCorrection
        CC.Enabled = true
        CC.Saturation = tonumber(config.Saturation) or 0
        CC.Contrast = tonumber(config.Contrast) or 0
        if config.TintEnabled then
            CC.TintColor = Color3.fromRGB(
                tonumber(config.TintR) or 255,
                tonumber(config.TintG) or 255,
                tonumber(config.TintB) or 255
            )
        else
            CC.TintColor = Color3.new(1, 1, 1)
        end

        -- ColorShift (Lighting)
        if config.ColorShiftEnabled then
            pcall(function()
                Lighting.ColorShift_Top = Color3.fromRGB(
                    tonumber(config.ColorShiftR) or 0,
                    tonumber(config.ColorShiftG) or 0,
                    tonumber(config.ColorShiftB) or 0
                )
                Lighting.ColorShift_Bottom = Lighting.ColorShift_Top
            end)
        end

        BloomFX.Enabled = config.BloomEnabled and true or false
        BloomFX.Intensity = tonumber(config.BloomIntensity) or 0.4
        BlurFX.Enabled = config.BlurEnabled and true or false
        BlurFX.Size = tonumber(config.BlurSize) or 8

        -- World mods (pesados: solo al toggle / apply)
        if config.WallXray then
            task.defer(function() pcall(applyWallXray, true) end)
        end
        if config.MapColorEnabled then
            task.defer(function() pcall(applyMapColor, true) end)
        end
        if config.RemoveTextures then
            task.defer(function() pcall(applyRemoveTextures, true) end)
        end
    end)
end

function ResetVisuals()
    pcall(function()
        Lighting.Brightness = oldLighting.Brightness
        Lighting.Ambient = oldLighting.Ambient
        Lighting.OutdoorAmbient = oldLighting.OutdoorAmbient
        Lighting.ExposureCompensation = oldLighting.ExposureCompensation
        Lighting.ClockTime = oldLighting.ClockTime
        Lighting.FogStart = oldLighting.FogStart
        Lighting.FogEnd = oldLighting.FogEnd
        Lighting.GlobalShadows = true
        pcall(function()
            Lighting.ColorShift_Top = Color3.new(0, 0, 0)
            Lighting.ColorShift_Bottom = Color3.new(0, 0, 0)
        end)
    end)
    CC.Enabled = false
    CC.TintColor = Color3.new(1, 1, 1)
    BloomFX.Enabled = false
    BlurFX.Enabled = false
    config.WallXray = false
    config.MapColorEnabled = false
    pcall(function() applyWallXray(false) end)
    pcall(function() applyMapColor(false) end)
    pcall(ResetCameraNormal)
end

-- ForceField visual (más transparente + estilos)
local FFHighlight = nil

function ClearForceFieldVisual(character)
    character = character or LocalPlayer.Character
    if FFHighlight then
        pcall(function() FFHighlight:Destroy() end)
        FFHighlight = nil
    end
    if character then
        for _, h in ipairs(character:GetChildren()) do
            if h:IsA("Highlight") and (h.Name == "ShakoFF" or h.Name == "ShakoFF2") then
                pcall(function() h:Destroy() end)
            end
        end
        for _, obj in pairs(character:GetDescendants()) do
            if obj:IsA("BasePart") and obj:GetAttribute("ShakoFFOrigMat") then
                pcall(function()
                    local matName = obj:GetAttribute("ShakoFFOrigMat")
                    local okMat, mat = pcall(function() return Enum.Material[matName] end)
                    if okMat and mat then obj.Material = mat end
                    local tr = obj:GetAttribute("ShakoFFOrigTrans")
                    if tr ~= nil then obj.Transparency = tr end
                    local col = obj:GetAttribute("ShakoFFOrigColor")
                    if typeof(col) == "Color3" then obj.Color = col end
                    obj:SetAttribute("ShakoFFOrigMat", nil)
                    obj:SetAttribute("ShakoFFOrigTrans", nil)
                    obj:SetAttribute("ShakoFFOrigColor", nil)
                end)
            end
        end
    end
end

-- Solo partes del cuerpo (sin Handle de accesorios = sin "cuadros")
local FF_BODY = {
    Head = true, Torso = true, UpperTorso = true, LowerTorso = true,
    LeftArm = true, RightArm = true, LeftLeg = true, RightLeg = true,
    ["Left Arm"] = true, ["Right Arm"] = true, ["Left Leg"] = true, ["Right Leg"] = true,
    LeftUpperArm = true, LeftLowerArm = true, LeftHand = true,
    RightUpperArm = true, RightLowerArm = true, RightHand = true,
    LeftUpperLeg = true, LeftLowerLeg = true, LeftFoot = true,
    RightUpperLeg = true, RightLowerLeg = true, RightFoot = true,
}

local function isBodyPartForFF(obj, character)
    if not obj or not obj:IsA("BasePart") then return false end
    if obj.Name == "HumanoidRootPart" then return false end
    -- nunca accesorios / tools (generan cajas ForceField)
    local p = obj.Parent
    while p and p ~= character do
        if p:IsA("Accessory") or p:IsA("Hat") or p:IsA("Tool") or p:IsA("Accoutrement") then
            return false
        end
        p = p.Parent
    end
    if FF_BODY[obj.Name] then return true end
    -- mesh del torso R6 a veces se llama distinto
    if obj.Parent == character and obj:IsA("MeshPart") and FF_BODY[obj.Name] then return true end
    return false
end

function ApplyForceField(character)
    character = character or LocalPlayer.Character
    if not character then return end
    if not config.ForceFieldVisual then
        ClearForceFieldVisual(character)
        return
    end

    local col = config.ForceFieldColor or Color3.fromRGB(180, 0, 255)
    local fillT = tonumber(config.ForceFieldTransparency)
    if fillT == nil then fillT = 0.55 end
    fillT = math.clamp(fillT, 0.05, 0.95)
    local outT = tonumber(config.ForceFieldOutlineTransparency)
    if outT == nil then outT = 0.7 end
    outT = math.clamp(outT, 0, 0.95)
    -- default: solo material en skin (sin highlight que "caja" el modelo)
    local style = tostring(config.ForceFieldStyle or "Material")

    -- opcional: ocultar accesorios para look limpio tipo kicia
    if config.ForceFieldRemoveAcc then
        for _, ch in ipairs(character:GetChildren()) do
            if ch:IsA("Accessory") or ch:IsA("Hat") or ch:IsA("Accoutrement") then
                pcall(function() ch:Destroy() end)
            end
        end
    end

    -- Highlight solo si se pide (Outline / Soft) — Occluded = sigue forma del mesh, no caja
    if style == "Highlight" or style == "Both" or style == "Soft" then
        local hl = character:FindFirstChild("ShakoFF")
        if not hl or not hl:IsA("Highlight") then
            if hl then pcall(function() hl:Destroy() end) end
            hl = Instance.new("Highlight")
            hl.Name = "ShakoFF"
            hl.DepthMode = Enum.HighlightDepthMode.Occluded
            hl.Parent = character
        end
        hl.FillColor = col
        hl.OutlineColor = col
        hl.FillTransparency = fillT
        hl.OutlineTransparency = outT
        hl.Enabled = true
        FFHighlight = hl
        if style == "Soft" then
            hl.FillTransparency = math.clamp(fillT + 0.2, 0.55, 0.95)
            hl.OutlineTransparency = math.clamp(outT + 0.15, 0.4, 0.95)
        end
    else
        local hl = character:FindFirstChild("ShakoFF")
        if hl then pcall(function() hl:Destroy() end) end
        FFHighlight = nil
    end

    -- Material ForceField SOLO en partes del cuerpo (no Handles de hats)
    if style == "Material" or style == "Both" or style == "Skin" then
        for _, obj in pairs(character:GetDescendants()) do
            if isBodyPartForFF(obj, character) then
                pcall(function()
                    if not obj:GetAttribute("ShakoFFOrigMat") then
                        obj:SetAttribute("ShakoFFOrigMat", obj.Material.Name)
                        obj:SetAttribute("ShakoFFOrigTrans", obj.Transparency)
                        obj:SetAttribute("ShakoFFOrigColor", obj.Color)
                    end
                    obj.Material = Enum.Material.ForceField
                    obj.Color = col
                    obj.Transparency = math.clamp(fillT * 0.7, 0.15, 0.85)
                end)
            end
        end
    else
        -- limpiar material si solo Highlight
        for _, obj in pairs(character:GetDescendants()) do
            if obj:IsA("BasePart") and obj:GetAttribute("ShakoFFOrigMat") then
                pcall(function()
                    local matName = obj:GetAttribute("ShakoFFOrigMat")
                    local okMat, mat = pcall(function() return Enum.Material[matName] end)
                    if okMat and mat then obj.Material = mat end
                    local tr = obj:GetAttribute("ShakoFFOrigTrans")
                    if tr ~= nil then obj.Transparency = tr end
                    local c = obj:GetAttribute("ShakoFFOrigColor")
                    if typeof(c) == "Color3" then obj.Color = c end
                    obj:SetAttribute("ShakoFFOrigMat", nil)
                    obj:SetAttribute("ShakoFFOrigTrans", nil)
                    obj:SetAttribute("ShakoFFOrigColor", nil)
                end)
            end
        end
    end
end

function StartForceField()
    config.ForceFieldVisual = true
    ApplyForceField(LocalPlayer.Character)
    SafeDisconnect("ForceField")
    SafeDisconnect("ForceFieldLoop")
    local char = LocalPlayer.Character
    if char then
        Connections.ForceField = char.ChildRemoved:Connect(function(ch)
            if config.ForceFieldVisual and (ch.Name == "ShakoFF" or ch.Name == "ShakoFF2") then
                task.defer(function() ApplyForceField(LocalPlayer.Character) end)
            end
        end)
    end
    Connections.ForceFieldLoop = task.spawn(function()
        local t = 0
        while config.ForceFieldVisual do
            task.wait(config.ForceFieldPulse and 0.08 or 1.5)
            if not config.ForceFieldVisual then break end
            if config.ForceFieldPulse then
                t = t + 0.08
                local base = tonumber(config.ForceFieldTransparency) or 0.72
                local pulse = base + math.sin(t * 3) * 0.08
                config.ForceFieldTransparency = math.clamp(pulse, 0.2, 0.95)
            end
            ApplyForceField(LocalPlayer.Character)
        end
    end)
    if not getgenv()._ShakoQuiet then
        Notify("ForceField", "ON")
    end
end

function StopForceField()
    config.ForceFieldVisual = false
    SafeDisconnect("ForceField")
    SafeDisconnect("ForceFieldLoop")
    ClearForceFieldVisual(LocalPlayer.Character)
    if not getgenv()._ShakoQuiet and not (getgenv()._ShakoBootAt and tick() - getgenv()._ShakoBootAt < 8) then
        Notify("ForceField", "OFF")
    end
end

function ApplyCharVisuals()
    -- NO tocar LocalTransparencyModifier (rompe zoom al apuntar)
    if not config.ApplyCharVisualsEnabled then return end
    local char = LocalPlayer.Character
    if not char then return end
    if config.RemoveAccessories then
        for _, p in pairs(char:GetChildren()) do
            if p:IsA("Accessory") or p:IsA("Hat") then p:Destroy() end
        end
    end
end

function MoveBodyTo(goal, lookAt)
    local myHRP = getHRP()
    if not myHRP then return end
    EnsureCameraFree()
    local current = myHRP.Position
    local delta = goal - current
    local dist = delta.Magnitude
    local function setAt(pos)
        if lookAt and not (config.RageDesync and config.RagebotEnabled) and not config.CameraIndependent then
            myHRP.CFrame = CFrame.new(pos, lookAt)
        else
            local flat = Vector3.new(goal.X - pos.X, 0, goal.Z - pos.Z)
            if flat.Magnitude > 0.15 then
                myHRP.CFrame = CFrame.new(pos, pos + flat)
            else
                myHRP.CFrame = CFrame.new(pos) * (myHRP.CFrame - myHRP.CFrame.Position)
            end
        end
        if config.ForceZeroVelocity then
            myHRP.AssemblyLinearVelocity = Vector3.zero
            myHRP.AssemblyAngularVelocity = Vector3.zero
        end
    end
    -- desync / SoftStick OFF = instant
    local forceInstant = (config.RageDesync and config.RagebotEnabled)
        or config.SoftStick == false
    local soft = config.SoftStick and not forceInstant
    if soft and dist > 0.08 then
        local maxStep = math.max(config.SoftStickMaxStep or 5, 0.5)
        local speed = config.SoftStickSpeed or 180
        local step = math.min(dist, maxStep, speed / 60)
        setAt(current + delta.Unit * step)
    else
        setAt(goal) -- instant
    end
end

function SafeEnemyCFrame(root, extraY, sideX, sideZ, lookAt)
    if not root then return end
    local base = root.Position
    local ox = (sideX or 0) + (config.StickOffsetX or 0)
    local oz = (sideZ or 0) + (config.StickOffsetZ or 0)
    local y = base.Y + (extraY or config.StickHeight)
    if config.LockYWhileStick then
        local myHRP = getHRP()
        if myHRP then y = myHRP.Position.Y end
    end
    local minY = workspace.FallenPartsDestroyHeight + (config.SafeZoneMinY or 30)
    if y < minY then y = minY + 1 end
    MoveBodyTo(Vector3.new(base.X + ox, y, base.Z + oz), lookAt)
end

-- ==================== TP ATRAS (INDEPENDIENTE DEL RAGE) ====================
function StartStickBehindTP()
    SafeDisconnect("StickBehind")
    config.StickBehindTP = true
    Connections.StickBehind = RunService.Heartbeat:Connect(function()
        if not config.StickBehindTP then return end
        if CanDoDangerousMove and not CanDoDangerousMove() then return end
        local my = getHRP and getHRP()
        if not my then return end
        local maxR = config.StickBehindRange
        if maxR == nil or maxR ~= maxR or maxR <= 0 then maxR = math.huge end -- nil/nan = infinito
        local best, bestD = nil, maxR
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr == LocalPlayer then continue end
            local char = plr.Character
            if not char then continue end
            local hum = char:FindFirstChildOfClass("Humanoid")
            local root = char:FindFirstChild("HumanoidRootPart")
            if not hum or not root or hum.Health <= 0 then continue end
            if IsEnemy and not IsEnemy(plr) then continue end
            local d = (my.Position - root.Position).Magnitude
            if d <= maxR and d < bestD then bestD, best = d, plr end
        end
        if not best or not best.Character then return end
        local root = best.Character:FindFirstChild("HumanoidRootPart")
        local head = best.Character:FindFirstChild("Head")
        if not root then return end
        local dist = tonumber(config.StickBehindDist) or 10
        local h = tonumber(config.StickBehindHeight) or 1.5
        local look = root.CFrame.LookVector
        local flat = Vector3.new(look.X, 0, look.Z)
        if flat.Magnitude > 0.05 then flat = flat.Unit else flat = Vector3.new(0, 0, 1) end
        local pos = root.Position - flat * dist + Vector3.new(0, h, 0)
        local minY = workspace.FallenPartsDestroyHeight + (config.SafeZoneMinY or 30)
        if pos.Y < minY then pos = Vector3.new(pos.X, minY + 1, pos.Z) end
        local lookAt = head and head.Position or (root.Position + Vector3.new(0, 1.5, 0))
        pcall(function()
            my.CFrame = CFrame.lookAt(pos, lookAt)
            my.AssemblyLinearVelocity = Vector3.zero
            my.AssemblyAngularVelocity = Vector3.zero
        end)
    end)
    Notify("TP Atras", "ON")
end

function StopStickBehindTP()
    config.StickBehindTP = false
    SafeDisconnect("StickBehind")
    Notify("TP Atras", "OFF")
end


function SyncAimOnShoot(headPos)
    return
end

function VoidRageOffset(root)
    local method = config.VoidRageMethods or "Side"
    local dist = config.VoidRageAwayDist or 100000000000
    local look = root.CFrame.LookVector
    local right = Vector3.new(-look.Z, 0, look.X)
    if right.Magnitude < 0.01 then right = Vector3.new(1, 0, 0) else right = right.Unit end
    local back = -Vector3.new(look.X, 0, look.Z)
    if back.Magnitude > 0.01 then back = back.Unit else back = Vector3.new(0, 0, 1) end
    local t = tick()

    if method == "Up" then
        return Vector3.new(0, dist, 0)
    elseif method == "Down" then
        return Vector3.new(0, -math.abs(dist) * 0.6, 0)
    elseif method == "Back" then
        return back * dist + Vector3.new(0, 4, 0)
    elseif method == "Front" then
        return -back * dist + Vector3.new(0, 3, 0)
    elseif method == "Diagonal" then
        local side = (math.floor(t * 2) % 2 == 0) and 1 or -1
        return (right * side + Vector3.new(0, 1, 0) + back * 0.5).Unit * dist
    elseif method == "Orbit" then
        local a = t * 8
        return Vector3.new(math.cos(a) * dist, 6 + math.sin(t * 3) * 4, math.sin(a) * dist)
    elseif method == "ZigZag" then
        local side = (math.floor(t * 5) % 2 == 0) and 1 or -1
        return right * dist * side + Vector3.new(0, 5 + (t % 2) * 4, 0)
    elseif method == "TeleportSpam" then
        local opts = {
            Vector3.new(0, dist, 0),
            right * dist,
            -right * dist,
            back * dist,
            Vector3.new(0, dist * 0.5, 0) + right * dist * 0.5,
        }
        return opts[math.random(1, #opts)]
    elseif method == "HighCircle" then
        local a = t * 7
        return Vector3.new(math.cos(a) * dist, dist * 0.7, math.sin(a) * dist)
    elseif method == "Random" then
        return Vector3.new((math.random() - 0.5) * dist * 2, math.random() * dist * 0.5, (math.random() - 0.5) * dist * 2)
    elseif method == "Circle" then
        local a = t * 6
        return Vector3.new(math.cos(a) * dist, 8, math.sin(a) * dist)
    else -- Side
        local side = (math.floor(t * 2) % 2 == 0) and 1 or -1
        return right * dist * side + Vector3.new(0, 6, 0)
    end
end

-- VOID SPAM
function computeVoidDrift(t)
    local nx, ny, nz, amp, freq = 0, 0, 0, 1, 0.0001
    for i = 1, 4 do
        nx = nx + math.noise(t * freq, 0, 0) * amp
        ny = ny + math.noise(0, t * freq, 0) * amp
        nz = nz + math.noise(0, 0, t * freq) * amp
        freq = freq * 2.37
        amp = amp * 0.5
    end
    local len = math.sqrt(nx * nx + ny * ny + nz * nz)
    if len < 0.001 then return Vector3.new(1, 0, 0) end
    return Vector3.new(nx / len, ny / len, nz / len)
end
function stepVoid(dt)
    local m, speed, radius = config.VoidMethod, config.VoidSpeed or 1e12, config.VoidRadius or 100000000000
    if m == "Drift" then
        voidDriftDir = voidDriftDir:Lerp(computeVoidDrift(voidElapsed), (config.VoidChaos or 0.98) * dt * 10)
        voidPos = voidPos + voidDriftDir * speed * dt
        if (voidPos - voidBase).Magnitude > radius then
            voidPos = voidBase + (voidPos - voidBase).Unit * radius
            voidDriftDir = -voidDriftDir
        end
        return voidPos
    elseif m == "Chaos" then
        local s = speed * dt * 5
        voidPos = voidPos + Vector3.new((math.random() - 0.5) * s, (math.random() - 0.5) * s, (math.random() - 0.5) * s)
        if (voidPos - voidBase).Magnitude > radius then voidPos = voidBase + (voidPos - voidBase).Unit * radius end
        return voidPos
    elseif m == "Loop" then
        local r = math.min(radius * 0.8, 1e9 + (voidElapsed % 100) * 1e7)
        return voidBase + Vector3.new(math.cos(voidElapsed * 2) * r, math.sin(voidElapsed * 1.3) * r * 0.2, math.sin(voidElapsed * 2) * r)
    elseif m == "Spiral" then
        local r = math.min(radius * 0.9, (voidElapsed % 50) * 2e9)
        return voidBase + Vector3.new(math.cos(voidElapsed * 3) * r, math.sin(voidElapsed * 0.5) * r * 0.3, math.sin(voidElapsed * 3) * r)
    else
        local r = radius * (math.random() > 0.5 and 1 or -1)
        return voidBase + Vector3.new(r, (math.random() - 0.5) * radius * 0.2, r)
            + Vector3.new((math.random() - 0.5) * radius, 0, (math.random() - 0.5) * radius)
    end
end
function voidEvadeCheck()
    if not config.VoidEvade or voidEvadeCD > 0 then return end
    local minD, threat = math.huge, Vector3.zero
    for _, p in pairs(Players:GetPlayers()) do
        if p == LocalPlayer or not p.Character then continue end
        local hrp = p.Character:FindFirstChild("HumanoidRootPart")
        local hum = p.Character:FindFirstChildOfClass("Humanoid")
        if hrp and hum and hum.Health > 0 then
            local pred = hrp.Position + hrp.AssemblyLinearVelocity * 0.5
            local d = (pred - intendedVoidPos).Magnitude
            if d < minD then minD = d threat = (pred - intendedVoidPos).Unit end
        end
    end
    if minD < (config.VoidEvadeRadius or 9e15) then
        local push = config.VoidEvadeDist or 100000000000
        voidPos = voidPos - threat * push
        if (voidPos - voidBase).Magnitude > (config.VoidRadius or 100000000000) then
            voidPos = voidBase + (voidPos - voidBase).Unit * (config.VoidRadius or 100000000000)
        end
        voidEvadeCD = config.VoidEvadeCooldown or 0.05
    end
end

-- Desync visual para TODOS los voids: otros te ven en el vacio, vos te ves quieto
function VoidServerMove(goal)
    local h = getHRP()
    if not h then return end
    local goalCF = typeof(goal) == "CFrame" and goal or CFrame.new(goal)
    local oldCF = h.CFrame
    local oldVel = h.AssemblyLinearVelocity
    local oldRot = h.AssemblyAngularVelocity
    pcall(function() RunService:UnbindFromRenderStep("__as_void_desync") end)
    h.CFrame = goalCF
    h.AssemblyLinearVelocity = Vector3.zero
    h.AssemblyAngularVelocity = Vector3.zero
    RunService:BindToRenderStep("__as_void_desync", Enum.RenderPriority.Camera.Value - 1, function()
        if h and h.Parent then
            h.CFrame = oldCF
            h.AssemblyLinearVelocity = oldVel
            h.AssemblyAngularVelocity = oldRot
        end
        pcall(function() RunService:UnbindFromRenderStep("__as_void_desync") end)
    end)
end

function StartVoidSpam()
    SafeDisconnect("VoidSpam")
    if not CanDoDangerousMove() then Notify("Void", "Espera spawn") return end
    local hrp = getHRP()
    local alt = config.VoidBaseAltitude or 100000000000
    if config.VoidEvade then
        alt = config.VoidEvadeDist or 100000000000
        config.VoidRadius = config.VoidRadius or 100000000000
    end
    if hrp then
        voidPos = Vector3.new(hrp.Position.X, alt, hrp.Position.Z)
        voidBase = Vector3.new(0, alt, 0)
        intendedVoidPos = voidPos
    end
    voidElapsed = 0
    Connections.VoidSpam = RunService.Heartbeat:Connect(function(dt)
        if not config.VoidSpamEnabled or not CanDoDangerousMove() or IsAntiUnnamedActive() then return end
        if config.VoidRage and config.RagebotEnabled then return end
        voidElapsed = voidElapsed + dt
        if voidEvadeCD > 0 then voidEvadeCD = voidEvadeCD - dt end
        voidEvadeCheck()
        intendedVoidPos = stepVoid(dt)
        local h = getHRP()
        if not h then return end
        VoidServerMove(CFrame.new(intendedVoidPos))
    end)
end
function StopVoidSpam()
    SafeDisconnect("VoidSpam")
    pcall(function() RunService:UnbindFromRenderStep("__as_void_desync") end)
end

function StartPredDodge()
    SafeDisconnect("PredDodge")
    Connections.PredDodge = RunService.Heartbeat:Connect(function(dt)
        if not config.PredDodgeEnabled or not CanDoDangerousMove() then return end
        if IsAntiUnnamedActive() or config.VoidSpamEnabled then return end
        if config.RagebotEnabled and config.VoidRage then return end
        local hrp, hum = getHRP(), getHum()
        if not hrp or not hum or hum.Health <= 0 then return end
        local myPos = hrp.Position
        for _, p in pairs(Players:GetPlayers()) do
            if p == LocalPlayer or not p.Character then continue end
            if config.TeamCheck and IsSameTeam(p) then continue end
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if not tr then continue end
            local vel = tr.AssemblyLinearVelocity
            if vel.Magnitude < (config.PredThreshold or 800) then continue end
            local pred = tr.Position + vel * (dt * 3 * (config.PredMult or 1))
            if (pred - myPos).Magnitude < (config.PredRadius or 20) and tick() - lastPredDodge > (config.PredCooldown or 0.3) then
                lastPredDodge = tick()
                local perp = Vector3.new(-vel.Z, 0, vel.X)
                if perp.Magnitude < 0.01 then perp = Vector3.new(1, 0, 0) end
                perp = perp.Unit
                local dir = math.random(0, 1) == 0 and perp or -perp
                local goal = myPos + dir * (config.PredDodgeDist or 30)
                if config.SoftStick then
                    local step = math.min((goal - myPos).Magnitude, config.SoftStickMaxStep or 5)
                    hrp.CFrame = CFrame.new(myPos + (goal - myPos).Unit * step)
                else
                    hrp.CFrame = CFrame.new(goal)
                end
                break
            end
        end
    end)
end
function StopPredDodge() SafeDisconnect("PredDodge") end

-- ==================== HVH DEFENSE ====================
local lastHurtHP = 100
local lastHurtEvade = 0
local lastNearEvade = 0
local lastMicro = 0
local defenseSide = 1

function defenseEnemyNear(maxDist)
    local my = getHRP()
    if not my then return nil, nil end
    local best, bestD, bestRoot = nil, maxDist, nil
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if config.TeamCheck and IsSameTeam(plr) then continue end
        local char = plr.Character
        if not char then continue end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root or hum.Health <= 0 then continue end
        local d = (my.Position - root.Position).Magnitude
        if d < bestD then bestD, best, bestRoot = d, plr, root end
    end
    return best, bestRoot
end

function pushAwayFrom(root, dist)
    local my = getHRP()
    if not my or not root then return end
    local dir = (my.Position - root.Position)
    if dir.Magnitude < 0.1 then
        dir = Vector3.new(math.random() - 0.5, 0, math.random() - 0.5)
    end
    dir = Vector3.new(dir.X, 0, dir.Z).Unit
    defenseSide = -defenseSide
    local side = Vector3.new(-dir.Z, 0, dir.X) * defenseSide
    local goal = my.Position + dir * dist + side * (dist * 0.35)
    local minY = workspace.FallenPartsDestroyHeight + 10
    if goal.Y < minY then goal = Vector3.new(goal.X, my.Position.Y, goal.Z) end
    my.CFrame = CFrame.new(goal)
    my.AssemblyLinearVelocity = (dir + side) * 40
end

function anyDefenseOn()
    return config.HurtEvade or config.NearEnemyEvade or config.AntiMeleeEnabled
        or config.AdaptiveDesync or config.VelocityBreak or config.RandomMicroMove
        or config.LowHPPanic or config.AntiUEClose or config.AntiUEPeek or config.DesyncSpam
end

function StartDefense()
    SafeDisconnect("Defense")
    local hum = getHum()
    if hum then lastHurtHP = hum.Health end
    local defFrame = 0
    Connections.Defense = RunService.Heartbeat:Connect(function()
        defFrame += 1
        if defFrame % 2 ~= 0 then return end -- OPT
        if not CanDoDangerousMove() then return end
        if IsAntiUnnamedActive() or config.VoidSpamEnabled then return end
        local my = getHRP()
        local hum = getHum()
        if not my or not hum then return end

        -- Velocity break (anti fling / knock fuerte de ragers)
        if config.VelocityBreak or config.AntiFling then
            local maxV = config.VelocityBreakMax or 140
            if my.AssemblyLinearVelocity.Magnitude > maxV then
                my.AssemblyLinearVelocity = Vector3.zero
                my.AssemblyAngularVelocity = Vector3.zero
            end
        end

        -- Hurt evade: al bajar HP, esquiva lateral
        if config.HurtEvade then
            if hum.Health < lastHurtHP - 1 then
                if tick() - lastHurtEvade >= (config.HurtEvadeCooldown or 0.35) then
                    lastHurtEvade = tick()
                    local _, root = defenseEnemyNear(80)
                    local dist = config.HurtEvadeDist or 18
                    if hum.Health <= (config.LowHPThreshold or 30) and config.LowHPPanic then
                        dist = config.LowHPBoostDist or 25
                    end
                    if root then
                        pushAwayFrom(root, dist)
                    else
                        defenseSide = -defenseSide
                        my.CFrame = my.CFrame + Vector3.new(defenseSide * dist, 2, 0)
                    end
                end
            end
            lastHurtHP = hum.Health
        end

        -- Cerca de melee / rager body
        if config.AntiMeleeEnabled or config.NearEnemyEvade then
            local range = config.AntiMeleeEnabled and (config.AntiMeleeRange or 7) or (config.NearEnemyDist or 8)
            local plr, root = defenseEnemyNear(range)
            if root and tick() - lastNearEvade > 0.2 then
                lastNearEvade = tick()
                local push = config.AntiMeleeEnabled and (config.AntiMeleePush or 16) or (config.NearEnemyPush or 14)
                pushAwayFrom(root, push)
            end
        end

        -- Adaptive desync: SOLO si toggle ON (antes se movia solo)
        if config.AdaptiveDesync == true and not config.Enabled and not config.RagebotEnabled then
            local _, root = defenseEnemyNear(config.AdaptiveDesyncRange or 40)
            if root and tick() - (lastMicro or 0) > 0.2 then
                lastMicro = tick()
                local p = math.min(config.AdaptiveDesyncPower or 6, 8)
                my.CFrame = my.CFrame * CFrame.new(
                    (math.random() - 0.5) * p * 0.04,
                    0,
                    (math.random() - 0.5) * p * 0.04
                )
            end
        end

        -- Anti UE close: si un rager se pega cerca, empuja LEJOS
        if config.AntiUEClose then
            local closest, croot = defenseEnemyNear(config.AntiUECloseDist or 35)
            if closest and croot then
                local d = (my.Position - croot.Position).Magnitude
                if d < (config.AntiUECloseDist or 35) then
                    pushAwayFrom(croot, config.AntiUEClosePush or 80)
                end
            end
        end

        -- Anti UE peek: side hop cuando hay enemigo en rango medio
        if config.AntiUEPeek then
            local _, root = defenseEnemyNear(config.AntiUEPeekDist or 40)
            if root and tick() - (lastHurtEvade or 0) > 0.4 then
                lastHurtEvade = tick()
                local dir = (my.Position - root.Position)
                if dir.Magnitude > 0.1 then
                    dir = Vector3.new(dir.X, 0, dir.Z).Unit
                    local side = Vector3.new(-dir.Z, 0, dir.X) * (math.random() > 0.5 and 1 or -1)
                    my.CFrame = CFrame.new(my.Position + side * 25 + dir * 10)
                    my.AssemblyLinearVelocity = side * 50
                end
            end
        end

        -- Desync spam anti-hit (micro jitter continuo)
        if config.DesyncSpam then
            local p = math.min(config.DesyncSpamPower or 8, 15)
            my.CFrame = my.CFrame * CFrame.new(
                (math.random() - 0.5) * p * 0.08,
                (math.random() - 0.5) * 0.3,
                (math.random() - 0.5) * p * 0.08
            )
        end

        -- Random micro move (solo toggle)
        if config.RandomMicroMove == true and tick() - lastMicro > 0.25 then
            lastMicro = tick()
            local p = config.RandomMicroPower or 3
            my.CFrame = my.CFrame + Vector3.new((math.random() - 0.5) * p, 0, (math.random() - 0.5) * p)
        end

        -- ===== ANTI-RAGER TACTICS =====
        local orbitOn = config.OrbitEvade or config.RagerCombo
        local heightOn = config.HeightJitter or config.RagerCombo
        local headOn = config.AntiHeadTP or config.RagerCombo
        local strafeOn = config.StrafePattern or config.RagerCombo
        local peekOn = config.FakePeek

        -- Orbit: gira alrededor del enemigo cercano (rompe TP a la cabeza)
        if orbitOn then
            local range = config.OrbitEvadeRange or 45
            local plr, root = defenseEnemyNear(range)
            if root then
                local radius = config.OrbitEvadeRadius or 12
                local spd = config.OrbitEvadeSpeed or 8
                local ang = tick() * spd
                local cx, cz = root.Position.X, root.Position.Z
                local y = root.Position.Y + 1.5
                local gx = cx + math.cos(ang) * radius
                local gz = cz + math.sin(ang) * radius
                local goal = Vector3.new(gx, y, gz)
                local cur = my.Position
                local step = (goal - cur)
                if step.Magnitude > 0.5 then
                    -- no saltar de golpe: interpolar
                    local maxStep = 4.5
                    if step.Magnitude > maxStep then step = step.Unit * maxStep end
                    my.CFrame = CFrame.new(cur + step)
                    my.AssemblyLinearVelocity = Vector3.new(step.X * 8, my.AssemblyLinearVelocity.Y, step.Z * 8)
                end
            end
        end

        -- Height jitter: sube/baja para que el head-aim falle
        if heightOn then
            local amt = config.HeightJitterAmount or 6
            local spd = config.HeightJitterSpeed or 12
            local oy = math.sin(tick() * spd) * amt
            local p = my.Position
            my.CFrame = CFrame.new(p.X, p.Y + oy * 0.08, p.Z) -- micro por frame
        end

        -- Anti head TP: si alguien está casi encima, empuja lejos
        if headOn then
            local dist = config.AntiHeadTPDist or 12
            local push = config.AntiHeadTPPush or 35
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr == LocalPlayer then continue end
                if config.TeamCheck and IsSameTeam and IsSameTeam(plr) then continue end
                local rh = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
                if not rh then continue end
                local delta = rh.Position - my.Position
                local horiz = Vector3.new(delta.X, 0, delta.Z).Magnitude
                if horiz < dist and delta.Y > 2 and delta.Y < 25 then
                    -- rager encima → lateral + atrás
                    local away = Vector3.new(-delta.X, 0, -delta.Z)
                    if away.Magnitude < 0.1 then away = my.CFrame.RightVector end
                    away = away.Unit
                    my.CFrame = my.CFrame + away * math.min(push * 0.15, 8) + Vector3.new(0, -1, 0)
                    my.AssemblyLinearVelocity = away * push
                    break
                end
            end
        end

        -- Strafe pattern A-D
        if strafeOn then
            local rate = config.StrafePatternRate or 0.18
            if not _strafeSide then _strafeSide = 1 end
            if not _lastStrafeT then _lastStrafeT = 0 end
            if tick() - _lastStrafeT >= rate then
                _lastStrafeT = tick()
                _strafeSide = -_strafeSide
                local d = config.StrafePatternDist or 10
                my.CFrame = my.CFrame + my.CFrame.RightVector * (_strafeSide * d * 0.35)
                local v = my.AssemblyLinearVelocity
                my.AssemblyLinearVelocity = Vector3.new(my.CFrame.RightVector.X * _strafeSide * 40, v.Y, my.CFrame.RightVector.Z * _strafeSide * 40)
            end
        end

        -- Fake peek: sale al lado y vuelve
        if peekOn then
            local rate = config.FakePeekRate or 0.55
            if not _lastPeekT then _lastPeekT = 0 end
            if not _peekPhase then _peekPhase = 0 end
            if tick() - _lastPeekT >= rate then
                _lastPeekT = tick()
                local d = config.FakePeekDist or 18
                if _peekPhase == 0 then
                    _peekOrigin = my.Position
                    my.CFrame = my.CFrame + my.CFrame.RightVector * d
                    _peekPhase = 1
                else
                    if _peekOrigin then
                        my.CFrame = CFrame.new(_peekOrigin.X, my.Position.Y, _peekOrigin.Z)
                    end
                    _peekPhase = 0
                end
            end
        end
    end)
end

function StopDefense()
    SafeDisconnect("Defense")
end

-- Defense NO auto-start (solo si usuario activa opciones)

-- ==================== HIT / KILL SOUNDS (Grief.cc assets + detection) ====================
local SoundService = game:GetService("SoundService")
local SFX = {
    -- rbxassetid from Grief.cc soundsassets (robados)
    hitSounds = {
        ["Rust HS"]           = 5043539486,
        ["Neverlose"]         = 97643101798871,
        ["Minecraft Bow"]     = 3442683707,
        ["Minecraft Hit"]     = 8766809464,
        ["CSGO"]              = 5764885315,
        ["Bubble"]            = 6534947588,
        ["Lazer"]             = 130791043,
        ["Pick"]              = 1347140027,
        ["Pop"]               = 198598793,
        ["Rust"]              = 1255040462,
        ["Sans"]              = 3188795283,
        ["Fart"]              = 130833677,
        ["Big"]               = 5332005053,
        ["Vine"]              = 5332680810,
        ["UwU"]               = 8679659744,
        ["Bruh"]              = 4578740568,
        ["Skeet"]             = 5633695679,
        ["Fatality"]          = 6534947869,
        ["Bonk"]              = 5766898159,
        ["Minecraft"]         = 5869422451,
        ["Gamesense"]         = 4817809188,
        ["RIFK7"]             = 9102080552,
        ["Bamboo"]            = 3769434519,
        ["Crowbar"]           = 546410481,
        ["Weeb"]              = 6442965016,
        ["Beep"]              = 8177256015,
        ["Bambi"]             = 8437203821,
        ["Stone"]             = 3581383408,
        ["Old Fatality"]      = 6607142036,
        ["Click"]             = 8053704437,
        ["Ding"]              = 7149516994,
        ["Snow"]              = 6455527632,
        ["Laser"]             = 7837461331,
        ["Mario"]             = 2815207981,
        ["Steve"]             = 4965083997,
        ["Call of Duty"]      = 5952120301,
        ["Bat"]               = 3333907347,
        ["TF2 Critical"]      = 296102734,
        ["Saber"]             = 8415678813,
        ["Baimware"]          = 3124331820,
        ["Osu"]               = 7149255551,
        ["TF2"]               = 2868331684,
        ["Slime"]             = 6916371803,
        ["Among Us"]          = 5700183626,
        ["One"]               = 7380502345,
        ["Soft Bell"]         = 9114487369,
        ["Minecraft Bow Hit"] = 1053296915,
        -- legacy aliases
        ["Never Lose"]        = 5043539486,
        ["TF2 Crit"]          = 296102734,
        ["Vine Boom"]         = 5332680810,
        ["MLG Airhorn"]       = 678089961,
        ["Boom Headshot"]     = 7361085557,
        ["Quake Hit"]         = 1455817260,
    },
    killSounds = {
        ["OOF"]           = 15666462,
        ["New Death"]     = 9118823103,
        ["Wake Up Bozo"]  = 9042248317,
        ["Emotional Dmg"] = 8388105535,
        ["You Are Bad"]   = 7163786825,
        ["Alone"]         = 7734793940,
        ["Fatality"]      = 6534947869,
        ["Vine"]          = 5332680810,
        ["Among Us"]      = 5700183626,
        ["TF2 Critical"]  = 296102734,
        ["Neverlose"]     = 97643101798871,
        ["Rust HS"]       = 5043539486,
    },
    hitNames = {
        "Rust HS","Neverlose","Gamesense","Skeet","Fatality","Bonk","Osu","CSGO",
        "TF2 Critical","TF2","Minecraft","Minecraft Hit","Bubble","Bruh","Vine",
        "Among Us","Click","Ding","Beep","Laser","Lazer","UwU","Weeb","Bamboo",
        "Crowbar","Bat","Saber","Stone","Snow","Steve","Mario","Call of Duty",
        "Baimware","RIFK7","Old Fatality","Big","Fart","Sans","Pick","Pop","Rust",
        "One","Soft Bell","Minecraft Bow","Minecraft Bow Hit","Bambi","Slime",
    },
    killNames = {
        "OOF","New Death","Wake Up Bozo","Emotional Dmg","You Are Bad","Alone",
        "Fatality","Vine","Among Us","TF2 Critical","Neverlose","Rust HS",
    },
    lastHit = 0, lastKill = 0, hp = {},
    auroraHooked = false,
    pitch = 1,
}

-- ==================== SFX PLAY (PlayLocalSound = sí se oye) ====================
-- IDs que casi siempre cargan en cliente Roblox
SFX.FALLBACK_HIT = 12222216
SFX.FALLBACK_KILL = 15666462
-- remap nombres populares a IDs que sí suenan
pcall(function()
    SFX.hitSounds = SFX.hitSounds or {}
    SFX.hitSounds["Rust HS"] = 12222216
    SFX.hitSounds["Neverlose"] = 12222216
    SFX.hitSounds["Never Lose"] = 12222216
    SFX.hitSounds["TF2 Critical"] = 296102734
    SFX.hitSounds["TF2 Crit"] = 296102734
    SFX.hitSounds["Click"] = 12222216
    SFX.hitSounds["CSGO"] = 5764885315
    SFX.hitSounds["Skeet"] = 5633695679
    SFX.hitSounds["Laser"] = 7837461331
    SFX.hitSounds["Lazer"] = 130791043
    SFX.killSounds = SFX.killSounds or {}
    SFX.killSounds["OOF"] = 15666462
    SFX.killSounds["Fatality"] = 6534947869
end)

function SFX.play(id, vol, pitch)
    local num = tonumber(tostring(id or ""):match("%d+")) or SFX.FALLBACK_HIT
    local volN = math.clamp(tonumber(vol) or tonumber(config and config.HitSoundVolume) or 7, 1, 10)
    local pid = math.clamp(tonumber(pitch) or tonumber(config and config.HitSoundPitch) or 1, 0.5, 3)

    local function one(soundId, volume)
        local s = Instance.new("Sound")
        s.Name = "ShakoHS"
        s.SoundId = "rbxassetid://" .. tostring(soundId)
        s.Volume = volume
        s.PlaybackSpeed = pid
        s.Looped = false
        s.PlayOnRemove = false
        -- parent al SoundService
        s.Parent = SoundService
        -- método oficial cliente
        pcall(function() SoundService:PlayLocalSound(s) end)
        pcall(function() s:Play() end)
        -- backup en PlayerGui (algunos juegos mutean SS)
        pcall(function()
            local pg = LocalPlayer:FindFirstChild("PlayerGui")
            if pg then
                local s2 = s:Clone()
                s2.Parent = pg
                pcall(function() SoundService:PlayLocalSound(s2) end)
                pcall(function() s2:Play() end)
                task.delay(4, function() pcall(function() s2:Destroy() end) end)
            end
        end)
        -- backup HRP
        pcall(function()
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local s3 = s:Clone()
                s3.Parent = hrp
                pcall(function() s3:Play() end)
                task.delay(4, function() pcall(function() s3:Destroy() end) end)
            end
        end)
        task.delay(5, function() pcall(function() s:Destroy() end) end)
        return s
    end

    one(num, volN)
    -- si el id no es el fallback, también no duplicar; el fallback se usa solo si falla detection
end

function SFX.hit()
    if not config.HitSoundEnabled then return end
    local now = tick()
    if now - (SFX.lastHit or 0) < (tonumber(config.HitSoundCooldown) or 0.08) then return end
    SFX.lastHit = now
    local name = config.SelectedHitSound or "Rust HS"
    if config.HitSoundMode == "Random" and SFX.hitNames and #SFX.hitNames > 0 then
        name = SFX.hitNames[math.random(1, #SFX.hitNames)]
    end
    local id = (SFX.hitSounds and SFX.hitSounds[name]) or SFX.FALLBACK_HIT
    SFX.play(id, config.HitSoundVolume or 7, config.HitSoundPitch or 1)
end

function SFX.kill()
    if not config.KillSoundEnabled then return end
    local now = tick()
    if now - (SFX.lastKill or 0) < (tonumber(config.KillSoundCooldown) or 0.15) then return end
    SFX.lastKill = now
    local name = config.SelectedKillSound or "OOF"
    if config.KillSoundMode == "Random" and SFX.killNames and #SFX.killNames > 0 then
        name = SFX.killNames[math.random(1, #SFX.killNames)]
    end
    local id = (SFX.killSounds and SFX.killSounds[name]) or SFX.FALLBACK_KILL
    SFX.play(id, config.KillSoundVolume or 8, config.KillSoundPitch or 1)
end

-- AURORA ClientViewModel hit sound hook (Rivals nativo)
task.spawn(function()
    task.wait(2)
    pcall(function()
        local path = LocalPlayer.PlayerScripts:FindFirstChild("Modules")
        path = path and path:FindFirstChild("ClientReplicatedClasses")
        path = path and path:FindFirstChild("ClientFighter")
        path = path and path:FindFirstChild("ClientItem")
        local vm_module = path and path:FindFirstChild("ClientViewModel")
        if not vm_module then return end
        local ClientViewModel = require(vm_module)
        if not ClientViewModel or not ClientViewModel.PlayHitmarkerSound then return end
        if SFX.auroraHooked then return end
        SFX.auroraHooked = true
        local orig = ClientViewModel.PlayHitmarkerSound
        ClientViewModel.PlayHitmarkerSound = function(self, is_crit, distance)
            local ctx = self.ClientItem and self.ClientItem.ClientFighter and self.ClientItem.ClientFighter.Player
            if ctx == LocalPlayer and config.HitSoundEnabled then
                local name = config.SelectedHitSound or "Neverlose"
                if config.HitSoundMode == "Random" then
                    name = SFX.hitNames[math.random(1, #SFX.hitNames)]
                end
                local id = SFX.hitSounds[name]
                if id then
                    pcall(function()
                        local vol = (config.HitSoundVolume or 2)
                        local pitch = (config.HitSoundPitch or 1)
                        if self._CreateHitmarkerSound then
                            local dist = tonumber(distance) or 1
                            if is_crit then
                                self:_CreateHitmarkerSound("rbxassetid://"..id, (3/math.max(dist,0.1)), pitch, vm_module, true, 1)
                                self:_CreateHitmarkerSound("rbxassetid://"..id, (2/math.max(dist,0.1)), pitch, vm_module, true, 1)
                            else
                                self:_CreateHitmarkerSound("rbxassetid://"..id, (1.5/math.max(dist,0.1)), pitch, vm_module, true, 1)
                            end
                        else
                            SFX.play(id, vol)
                        end
                    end)
                    return
                end
            end
            return orig(self, is_crit, distance)
        end
        print("[shako] AURORA hitmarker hooked")
    end)
end)

-- Detección de hit: HealthChanged (fiable) + poll de respaldo
local function sfxOnDamage(plr, newHp, oldHp)
    if not plr or plr == LocalPlayer then return end
    if not (config.HitSoundEnabled or config.KillSoundEnabled) then return end
    oldHp = tonumber(oldHp) or SFX.hp[plr]
    newHp = tonumber(newHp)
    if newHp == nil then return end
    -- ignorar micro cambios / regeneración / sync al equipar
    if oldHp == nil then
        SFX.hp[plr] = newHp
        return
    end
    local drop = oldHp - newHp
    -- kill
    if oldHp > 0 and newHp <= 0 then
        if config.KillSoundEnabled then SFX.kill() end
        SFX.hp[plr] = newHp
        return
    end
    -- hit real: bajó al menos ~2 HP (evita ruido de weapon switch / replicate)
    if drop >= 2 then
        if config.HitSoundEnabled then SFX.hit() end
    end
    SFX.hp[plr] = newHp
end

local function sfxHookHumanoid(plr, hum)
    if not hum or SFX._humHooked and SFX._humHooked[hum] then return end
    SFX._humHooked = SFX._humHooked or setmetatable({}, { __mode = "k" })
    SFX._humHooked[hum] = true
    SFX.hp[plr] = hum.Health
    hum.HealthChanged:Connect(function(hp)
        sfxOnDamage(plr, hp, SFX.hp[plr])
    end)
end

local function sfxHookPlayer(plr)
    if plr == LocalPlayer then return end
    plr.CharacterAdded:Connect(function(char)
        task.defer(function()
            local hum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 3)
            if hum then sfxHookHumanoid(plr, hum) end
        end)
    end)
    if plr.Character then
        local hum = plr.Character:FindFirstChildOfClass("Humanoid")
        if hum then sfxHookHumanoid(plr, hum) end
    end
end

for _, plr in ipairs(Players:GetPlayers()) do sfxHookPlayer(plr) end
Players.PlayerAdded:Connect(sfxHookPlayer)
Players.PlayerRemoving:Connect(function(p) SFX.hp[p] = nil end)

-- poll de respaldo (por si HealthChanged no dispara en algún caso)
task.spawn(function()
    while true do
        task.wait(0.12)
        if not (config.HitSoundEnabled or config.KillSoundEnabled) then continue end
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr == LocalPlayer then continue end
            local hum = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
            if not hum then continue end
            local prev = SFX.hp[plr]
            local hp = hum.Health
            if prev ~= nil and hp ~= prev then
                sfxOnDamage(plr, hp, prev)
            else
                SFX.hp[plr] = hp
            end
        end
    end
end)


-- GUI watchers DESACTIVADOS: al cambiar arma Rivals crea UI con números
-- y disparaba hit sound sin parar. Solo usamos:
-- 1) AURORA PlayHitmarkerSound (hit real del juego)
-- 2) HealthChanged en enemigos (bajó vida de verdad)

-- re-track HP on CharacterAdded (respawn)
Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        task.wait(0.3)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then SFX.hp[plr] = hum.Health end
    end)
end)
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        plr.CharacterAdded:Connect(function(char)
            task.wait(0.3)
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then SFX.hp[plr] = hum.Health end
        end)
    end
end


-- ==================== HIT FEEDBACK & TRACERS (LuaHook-style) ====================
local HitFX = { folder = nil, hmGui = nil, bannerGui = nil, lastBanner = 0 }

local function hitFxColor(key, default)
    local c = config[key]
    if typeof(c) == "Color3" then return c end
    return default
end

local function ensureHitFxFolder()
    if HitFX.folder and HitFX.folder.Parent then return HitFX.folder end
    local f = Instance.new("Folder")
    f.Name = "ShakoHitFX"
    f.Parent = workspace
    HitFX.folder = f
    return f
end

function HitFX.spawnBeam(muzzlePos, hitPos)
    if not config.FXBeamTracer then return end
    if not muzzlePos or not hitPos then return end
    local dist = (hitPos - muzzlePos).Magnitude
    if dist < 0.5 or dist > 2000 then return end
    local style = tostring(config.FXBeamStyle or "Line")
    local color = hitFxColor("FXBeamHitColor", Color3.fromRGB(255, 194, 75))
    local dur = tonumber(config.FXBeamDuration) or 0.35
    local thick = tonumber(config.FXBeamThickness) or 0.12
    local folder = ensureHitFxFolder()
    local mid = (muzzlePos + hitPos) * 0.5
    local part = Instance.new("Part")
    part.Name = "ShakoBeam"
    part.Anchored = true
    part.CanCollide = false
    part.CanQuery = false
    part.CanTouch = false
    part.CastShadow = false
    part.Material = (style == "Glow" or style == "Prism") and Enum.Material.Neon or Enum.Material.SmoothPlastic
    part.Color = color
    part.Transparency = style == "Line" and 0.15 or 0.05
    part.Size = Vector3.new(thick, thick, dist)
    part.CFrame = CFrame.lookAt(mid, hitPos)
    part.Parent = folder
    if style == "Glow" or style == "Prism" then
        local a0 = Instance.new("Attachment"); a0.Parent = part
        local a1 = Instance.new("Attachment"); a1.Parent = part
        a0.Position = Vector3.new(0, 0, -dist/2)
        a1.Position = Vector3.new(0, 0, dist/2)
        local beam = Instance.new("Beam")
        beam.Attachment0 = a0
        beam.Attachment1 = a1
        beam.Color = ColorSequence.new(color)
        beam.Width0 = thick * 4
        beam.Width1 = thick * 1.2
        beam.LightEmission = 1
        beam.FaceCamera = true
        beam.Parent = part
        if style == "Prism" then
            beam.Texture = "rbxassetid://446111271"
            beam.TextureSpeed = 2
            beam.TextureLength = 4
        end
    end
    if style == "Arc" then
        part.Size = Vector3.new(thick * 1.5, thick * 1.5, dist)
        part.Transparency = 0.25
    end
    task.delay(dur, function() pcall(function() part:Destroy() end) end)
end

function HitFX.triggerMarker(crit, lethal)
    if not config.FXHitMarker then return end
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return end
    local gui = HitFX.hmGui
    if not gui or not gui.Parent then
        gui = Instance.new("ScreenGui")
        gui.Name = "ShakoHitMarker"
        gui.IgnoreGuiInset = true
        gui.ResetOnSpawn = false
        gui.DisplayOrder = 100
        gui.Parent = pg
        HitFX.hmGui = gui
    end
    for _, c in ipairs(gui:GetChildren()) do c:Destroy() end
    local color = hitFxColor("FXHitMarkerColor", Color3.fromRGB(255, 255, 255))
    if crit then color = hitFxColor("FXHitMarkerCritColor", Color3.fromRGB(255, 194, 75)) end
    if lethal then color = hitFxColor("FXHitMarkerLethalColor", Color3.fromRGB(255, 64, 78)) end
    local style = tostring(config.FXHitMarkerStyle or "X")
    local gap = tonumber(config.FXHitMarkerGap) or 5
    local len = tonumber(config.FXHitMarkerLen) or 8
    local th = tonumber(config.FXHitMarkerThickness) or 2
    local function line(x1, y1, x2, y2)
        local f = Instance.new("Frame")
        f.BackgroundColor3 = color
        f.BorderSizePixel = 0
        f.AnchorPoint = Vector2.new(0.5, 0.5)
        local dx, dy = x2 - x1, y2 - y1
        local dist = math.sqrt(dx*dx + dy*dy)
        f.Size = UDim2.new(0, math.max(dist, 1), 0, th)
        f.Position = UDim2.new(0.5, (x1+x2)/2, 0.5, (y1+y2)/2)
        f.Rotation = math.deg(math.atan2(dy, dx))
        f.Parent = gui
        return f
    end
    local frames = {}
    if style == "Circle" then
        local r = Instance.new("Frame")
        r.BackgroundTransparency = 1
        r.Size = UDim2.new(0, (gap+len)*2, 0, (gap+len)*2)
        r.AnchorPoint = Vector2.new(0.5, 0.5)
        r.Position = UDim2.new(0.5, 0, 0.5, 0)
        r.Parent = gui
        local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(1, 0); c.Parent = r
        local s = Instance.new("UIStroke"); s.Color = color; s.Thickness = th; s.Parent = r
        frames[#frames+1] = r
    elseif style == "Plus" then
        frames[#frames+1] = line(0, -(gap+len), 0, -gap)
        frames[#frames+1] = line(0, gap, 0, gap+len)
        frames[#frames+1] = line(-(gap+len), 0, -gap, 0)
        frames[#frames+1] = line(gap, 0, gap+len, 0)
    else
        local d = 0.7071
        frames[#frames+1] = line(-d*(gap+len), -d*(gap+len), -d*gap, -d*gap)
        frames[#frames+1] = line(d*gap, d*gap, d*(gap+len), d*(gap+len))
        frames[#frames+1] = line(d*(gap+len), -d*(gap+len), d*gap, -d*gap)
        frames[#frames+1] = line(-d*gap, d*gap, -d*(gap+len), d*(gap+len))
    end
    task.delay(0.22, function()
        for _, f in ipairs(frames) do pcall(function() f:Destroy() end) end
    end)
end

function HitFX.pushDamageNumber(worldPos, dmg, crit, lethal)
    if not config.FXDamageNumbers then return end
    if not worldPos then return end
    local p = Instance.new("Part")
    p.Anchored = true; p.CanCollide = false; p.Transparency = 1
    p.Size = Vector3.new(0.2,0.2,0.2)
    p.Position = worldPos + Vector3.new(0, 1.2, 0)
    p.Parent = ensureHitFxFolder()
    local bill = Instance.new("BillboardGui")
    bill.Size = UDim2.new(0, 80, 0, 40)
    bill.StudsOffset = Vector3.new(math.random(-8,8)/10, 0.5, 0)
    bill.AlwaysOnTop = true
    bill.Adornee = p
    bill.Parent = p
    local t = Instance.new("TextLabel")
    t.BackgroundTransparency = 1
    t.Size = UDim2.new(1,0,1,0)
    t.Font = Enum.Font.GothamBold
    t.TextSize = crit and 22 or 18
    t.TextStrokeTransparency = 0.4
    t.Text = tostring(math.floor((dmg or 0) + 0.5))
    if lethal then
        t.TextColor3 = hitFxColor("FXHitMarkerLethalColor", Color3.fromRGB(255, 64, 78))
    elseif crit then
        t.TextColor3 = hitFxColor("FXHitMarkerCritColor", Color3.fromRGB(255, 194, 75))
    else
        t.TextColor3 = Color3.fromRGB(255, 255, 255)
    end
    t.Parent = bill
    task.spawn(function()
        for i = 1, 20 do
            p.Position = p.Position + Vector3.new(0, 0.08, 0)
            t.TextTransparency = i / 22
            t.TextStrokeTransparency = 0.4 + i/30
            task.wait(0.03)
        end
        pcall(function() p:Destroy() end)
    end)
end

function HitFX.triggerKillBanner(name)
    if not config.FXKillBanner then return end
    if tick() - HitFX.lastBanner < 0.4 then return end
    HitFX.lastBanner = tick()
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return end
    if HitFX.bannerGui then pcall(function() HitFX.bannerGui:Destroy() end) end
    local gui = Instance.new("ScreenGui")
    gui.Name = "ShakoKillBanner"
    gui.IgnoreGuiInset = true
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 120
    gui.Parent = pg
    HitFX.bannerGui = gui
    local lab = Instance.new("TextLabel")
    lab.BackgroundTransparency = 1
    lab.Size = UDim2.new(0.5, 0, 0, 36)
    lab.Position = UDim2.new(0.25, 0, 0.18, 0)
    lab.Font = Enum.Font.GothamBold
    lab.TextSize = 28
    lab.TextColor3 = hitFxColor("FXKillBannerColor", Color3.fromRGB(255, 194, 75))
    lab.TextStrokeTransparency = 0.3
    lab.Text = "ELIMINATED  " .. tostring(name or "")
    lab.Parent = gui
    task.spawn(function()
        for i = 1, 15 do
            lab.TextTransparency = i / 18
            lab.TextStrokeTransparency = 0.3 + i/25
            task.wait(0.05)
        end
        pcall(function() gui:Destroy() end)
        if HitFX.bannerGui == gui then HitFX.bannerGui = nil end
    end)
end

function HitFX.onHit(plr, dmg, wasKill, hitPos)
    local crit = (tonumber(dmg) or 0) >= 40
    local lethal = wasKill and true or false
    HitFX.triggerMarker(crit, lethal)
    if hitPos then
        HitFX.pushDamageNumber(hitPos, dmg or 0, crit, lethal)
        local cam = workspace.CurrentCamera
        if cam and config.FXBeamTracer then
            local origin = cam.CFrame.Position + cam.CFrame.RightVector * 1.2 - cam.CFrame.UpVector * 0.9 + cam.CFrame.LookVector * 1.2
            HitFX.spawnBeam(origin, hitPos)
        end
    elseif config.FXBeamTracer and plr and plr.Character then
        local cam = workspace.CurrentCamera
        local head = plr.Character:FindFirstChild("Head")
        if cam and head then
            HitFX.spawnBeam(cam.CFrame.Position + cam.CFrame.LookVector * 2, head.Position)
        end
    end
    if lethal and plr then
        HitFX.triggerKillBanner(plr.DisplayName or plr.Name)
    end
end

-- HP drop detector (OPT: cada 0.1s, no cada frame)
task.spawn(function()
    task.wait(1)
    local last = {}
    while true do
        task.wait(0.1)
        if not (config.FXHitMarker or config.FXBeamTracer or config.FXDamageNumbers or config.FXKillBanner) then
            continue
        end
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr == LocalPlayer then continue end
            local char = plr.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if not hum then continue end
            local hp = hum.Health
            local prev = last[plr]
            if prev and hp < prev - 0.5 then
                local dmg = prev - hp
                local head = char:FindFirstChild("Head")
                HitFX.onHit(plr, dmg, hp <= 0 and prev > 0, head and head.Position)
            end
            last[plr] = hp
        end
    end
end)


-- ==================== SPOOFER (nombres + perfil UI) ====================
local Spoof = {
    map = {},
    randomPool = {
        "Guest","Player","User","Unknown","Hidden","Shadow","Ghost","Anon",
        "xXProXx","Noob","Bot","NPC","???","-----","null","error",
    },
    tagged = {}, -- TextLabels ya tocados [obj] = true
}

-- ==================== DEVICE SPOOFER ====================
-- ==================== DEVICE SPOOFER (Mobile / Console / VR / PC) ====================
-- Spoof local de flags que leen scripts del juego. No garantiza el server de Roblox.
local DeviceSpoof = {
    hooked = false,
    conn = nil,
}

local DEVICE_PRESETS = {
    PC = {
        Platform = Enum.Platform.Windows,
        TouchEnabled = false,
        KeyboardEnabled = true,
        MouseEnabled = true,
        GamepadEnabled = false,
        GyroscopeEnabled = false,
        AccelerometerEnabled = false,
        VREnabled = false,
        TenFoot = false,
        label = "PC",
        LastInput = Enum.UserInputType.Keyboard,
    },
    Mobile = {
        Platform = Enum.Platform.IOS,
        TouchEnabled = true,
        KeyboardEnabled = false,
        MouseEnabled = false,
        GamepadEnabled = false,
        GyroscopeEnabled = true,
        AccelerometerEnabled = true,
        VREnabled = false,
        TenFoot = false,
        label = "Mobile",
        LastInput = Enum.UserInputType.Touch,
    },
    Console = {
        Platform = Enum.Platform.XBoxOne,
        TouchEnabled = false,
        KeyboardEnabled = false,
        MouseEnabled = false,
        GamepadEnabled = true,
        GyroscopeEnabled = false,
        AccelerometerEnabled = false,
        VREnabled = false,
        TenFoot = true,
        label = "Console",
        LastInput = Enum.UserInputType.Gamepad1,
    },
    Controller = {
        Platform = Enum.Platform.Windows,
        TouchEnabled = false,
        KeyboardEnabled = true,
        MouseEnabled = true,
        GamepadEnabled = true,
        GyroscopeEnabled = false,
        AccelerometerEnabled = false,
        VREnabled = false,
        TenFoot = false,
        label = "Controller",
        LastInput = Enum.UserInputType.Gamepad1,
    },
    VR = {
        Platform = Enum.Platform.Windows,
        TouchEnabled = false,
        KeyboardEnabled = true,
        MouseEnabled = true,
        GamepadEnabled = true,
        GyroscopeEnabled = true,
        AccelerometerEnabled = true,
        VREnabled = true,
        TenFoot = false,
        label = "VR",
        LastInput = Enum.UserInputType.Gamepad1,
    },
}

local UIS_KEYS = {
    TouchEnabled = true,
    KeyboardEnabled = true,
    MouseEnabled = true,
    GamepadEnabled = true,
    GyroscopeEnabled = true,
    AccelerometerEnabled = true,
    VREnabled = true,
}

function DeviceSpoof.getPreset()
    return DEVICE_PRESETS[config.DeviceSpoofType] or DEVICE_PRESETS.Mobile
end

function DeviceSpoof.applyAttributes()
    local p = DeviceSpoof.getPreset()
    pcall(function()
        LocalPlayer:SetAttribute("Device", p.label)
        LocalPlayer:SetAttribute("Platform", p.label)
        LocalPlayer:SetAttribute("InputType", p.label)
        LocalPlayer:SetAttribute("IsMobile", p.TouchEnabled == true)
        LocalPlayer:SetAttribute("IsVR", p.VREnabled == true)
        LocalPlayer:SetAttribute("IsConsole", config.DeviceSpoofType == "Console")
        LocalPlayer:SetAttribute("IsController", p.GamepadEnabled == true)
        LocalPlayer:SetAttribute("IsPC", config.DeviceSpoofType == "PC")
        LocalPlayer:SetAttribute("DeviceType", p.label)
        LocalPlayer:SetAttribute("PreferredInput", p.label)
    end)
    -- carpeta señal local por si el juego la lee
    pcall(function()
        local folder = RS:FindFirstChild("__DeviceSpoof") or Instance.new("Folder")
        folder.Name = "__DeviceSpoof"
        folder.Parent = RS
        folder:SetAttribute("Type", p.label)
        folder:SetAttribute("Touch", p.TouchEnabled)
        folder:SetAttribute("Gamepad", p.GamepadEnabled)
        folder:SetAttribute("VR", p.VREnabled)
        folder:SetAttribute("TenFoot", p.TenFoot)
    end)
end

function DeviceSpoof.isUIS(obj)
    if not obj then return false end
    if obj == UserInputService then return true end
    local ok, cn = pcall(function() return obj.ClassName end)
    return ok and cn == "UserInputService"
end

function DeviceSpoof.isGuiService(obj)
    if not obj then return false end
    local ok, cn = pcall(function() return obj.ClassName end)
    return ok and cn == "GuiService"
end

function DeviceSpoof.isVRService(obj)
    if not obj then return false end
    local ok, cn = pcall(function() return obj.ClassName end)
    return ok and cn == "VRService"
end

function DeviceSpoof.hook()
    if DeviceSpoof.hooked then
        DeviceSpoof.applyAttributes()
        return true
    end
    if not hookmetamethod or not getrawmetatable then
        DeviceSpoof.applyAttributes()
        return false
    end

    local success = false
    pcall(function()
        local mt = getrawmetatable(game)
        if not mt then return end
        pcall(function() setreadonly(mt, false) end)

        local oldIndex = mt.__index
        local oldNamecall = mt.__namecall

        -- cache servicios una vez (no GetService cada index)
        local UIS = UserInputService
        local GuiS = game:GetService("GuiService")
        local VRS = nil
        pcall(function() VRS = game:GetService("VRService") end)

        local function resolveIndex(self, key)
            if not config.DeviceSpoofEnabled then
                return oldIndex(self, key)
            end
            -- comparacion barata por identidad
            if self == UIS and type(key) == "string" and UIS_KEYS[key] then
                return DeviceSpoof.getPreset()[key]
            end
            if self == GuiS and key == "IsTenFootInterface" then
                return DeviceSpoof.getPreset().TenFoot
            end
            if VRS and self == VRS and key == "VREnabled" then
                return DeviceSpoof.getPreset().VREnabled
            end
            return oldIndex(self, key)
        end

        mt.__index = newcclosure and newcclosure(resolveIndex) or resolveIndex

        local function resolveNamecall(self, ...)
            local method = getnamecallmethod and getnamecallmethod() or ""
            if config.DeviceSpoofEnabled then
                local p = DeviceSpoof.getPreset()
                if DeviceSpoof.isUIS(self) then
                    if method == "GetPlatform" then return p.Platform end
                    if method == "GetLastInputType" then return p.LastInput end
                    if method == "GetConnectedGamepads" then
                        if p.GamepadEnabled then
                            return { Enum.UserInputType.Gamepad1 }
                        end
                        return {}
                    end
                    if method == "GetStringForKeyCode" then
                        return oldNamecall(self, ...)
                    end
                end
                if DeviceSpoof.isGuiService(self) and method == "IsTenFootInterface" then
                    return p.TenFoot
                end
                if DeviceSpoof.isVRService(self) and (method == "GetPropertyChangedSignal" or method == "") then
                    -- fall through
                end
                if DeviceSpoof.isVRService(self) and method == "GetUserCFrame" then
                    return oldNamecall(self, ...)
                end
            end
            return oldNamecall(self, ...)
        end

        mt.__namecall = newcclosure and newcclosure(resolveNamecall) or resolveNamecall

        pcall(function() setreadonly(mt, true) end)
        success = true
        DeviceSpoof.hooked = true
    end)

    -- refuerzo: hook directo de propiedades via hookfunction si existe
    pcall(function()
        if not hookfunction then return end
        -- no siempre existe; attributes + metamethod suelen bastar
    end)

    DeviceSpoof.applyAttributes()

    -- keep-alive LENTO (no cada frame = no lag)
    if DeviceSpoof.conn then pcall(function() DeviceSpoof.conn:Disconnect() end) end
    DeviceSpoof.conn = nil
    task.spawn(function()
        while config.DeviceSpoofEnabled do
            DeviceSpoof.applyAttributes()
            task.wait(5)
        end
    end)

    return success
end

function DeviceSpoof.enable()
    config.DeviceSpoofEnabled = true
    local ok = true
    if not DeviceSpoof.hooked then
        ok = DeviceSpoof.hook()
    else
        DeviceSpoof.applyAttributes()
    end
    local p = DeviceSpoof.getPreset()
    Notify("Device", "Spoof → " .. p.label .. (ok and " (hook OK)" or " (atributos; executor sin hookmetamethod)"))
end

function DeviceSpoof.disable()
    config.DeviceSpoofEnabled = false
    if DeviceSpoof.conn then
        pcall(function() DeviceSpoof.conn:Disconnect() end)
        DeviceSpoof.conn = nil
    end
    pcall(function()
        LocalPlayer:SetAttribute("Device", nil)
        LocalPlayer:SetAttribute("Platform", nil)
        LocalPlayer:SetAttribute("InputType", nil)
        LocalPlayer:SetAttribute("IsMobile", nil)
        LocalPlayer:SetAttribute("IsVR", nil)
        LocalPlayer:SetAttribute("IsConsole", nil)
    end)
    Notify("Device", "OFF (hook queda hasta rejoin)")
end

function Spoof.getName(plr)
    if not config.SpoofEnabled or not plr then
        return plr and (plr.DisplayName ~= "" and plr.DisplayName or plr.Name) or "?"
    end
    if plr == LocalPlayer and config.SpoofMyName and config.SpoofMyNameValue ~= "" then
        return tostring(config.SpoofMyNameValue)
    end
    if Spoof.map[plr] and Spoof.map[plr] ~= "" then
        return Spoof.map[plr]
    end
    if config.SpoofTargetName ~= "" and config.SpoofTargetFake ~= "" then
        local real = string.lower(plr.Name)
        local disp = string.lower(plr.DisplayName or "")
        local want = string.lower(config.SpoofTargetName)
        if real:find(want, 1, true) or disp:find(want, 1, true) then
            return tostring(config.SpoofTargetFake)
        end
    end
    if plr ~= LocalPlayer and config.SpoofOthers then
        if config.SpoofRandomOthers then
            if not Spoof.map[plr] then
                Spoof.map[plr] = Spoof.randomPool[math.random(1, #Spoof.randomPool)] .. math.random(10, 99)
            end
            return Spoof.map[plr]
        end
        if config.SpoofOthersValue ~= "" then
            return tostring(config.SpoofOthersValue)
        end
    end
    return plr.DisplayName ~= "" and plr.DisplayName or plr.Name
end
function Spoof.getMyUser()
    local u = config.SpoofMyUserValue
    if u and u ~= "" then return tostring(u) end
    return tostring(config.SpoofMyNameValue or "shako.win")
end
function Spoof.setPlayer(plr, fake)
    if not plr then return end
    if fake and fake ~= "" then Spoof.map[plr] = tostring(fake) else Spoof.map[plr] = nil end
end
function Spoof.clearAll()
    table.clear(Spoof.map)
end
function Spoof.applyNametags(char, fakeName)
    if not char or not fakeName then return end
    -- SOLO labels de nombre. NUNCA racha / stats / billboards genéricos
    -- (antes pisaba TODO y salía "Shako.win Shako.win" + racha con el name)
    local allowName = {
        DisplayName = true, PlayerName = true, NameLabel = true,
        Username = true, Handle = true, HeaderText = true,
    }
    pcall(function()
        for _, v in ipairs(char:GetDescendants()) do
            if not (v:IsA("TextLabel") or v:IsA("TextButton")) then continue end
            local on = v.Name or ""
            local pn = v.Parent and v.Parent.Name or ""
            local lowerN = string.lower(on .. " " .. pn)
            -- bloquear racha / stats
            if lowerN:find("streak", 1, true) or lowerN:find("racha", 1, true)
                or lowerN:find("win", 1, true) or lowerN:find("kill", 1, true)
                or lowerN:find("level", 1, true) or lowerN:find("elo", 1, true)
                or lowerN:find("score", 1, true) or lowerN:find("stat", 1, true)
                or on == "Value" or on == "Amount" or on == "Count" then
                continue
            end
            if not allowName[on] and pn ~= "NameContainer" and pn ~= "Title" and pn ~= "Subtitle" then
                continue
            end
            local t = v.Text or ""
            if t == "" or t:match("^%s*[%d%.,%%]+%s*$") then continue end
            -- no tocar si ya es el fake (evita doble escritura)
            if string.lower(t) == string.lower(fakeName) then continue end
            if string.lower(t) == "@" .. string.lower(fakeName) then continue end
            -- Username/Handle → @user si hay spoof de user, si no dejar
            if on == "Username" or on == "Handle" or pn == "Subtitle" then
                local fu = Spoof.getMyUser and Spoof.getMyUser() or fakeName
                if t:sub(1,1) == "@" or on == "Handle" then
                    v.Text = "@" .. tostring(fu):gsub("^@", "")
                end
                continue
            end
            v.Text = fakeName
        end
    end)
end
-- reescribe texto en GUIs: perfil, leaderboard, lista jugadores, top bar
function Spoof.patchTextObject(obj, realName, realDisplay, fakeDisplay, fakeUser)
    if not obj then return end
    if not (obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox")) then return end
    local t = obj.Text
    if not t or t == "" then return end
    -- solo numeros / score puro → no tocar
    if t:match("^%s*[%d%.,%%]+%s*$") then return end
    local lower = string.lower(t)
    local skip = false
    for _, w in ipairs({"win rate", "wins", "level", "platinum", "played", "armas", "fusil", "revolver", "granada"}) do
        if lower:find(w, 1, true) then skip = true break end
    end
    if skip then return end

    local rn = realName and string.lower(realName) or ""
    local rd = realDisplay and string.lower(realDisplay) or ""
    local fd = fakeDisplay or ""
    local fu = fakeUser or fd
    if fd == "" then return end
    -- si ya es el nombre falso, dejar
    if lower == string.lower(fd) or lower == string.lower(fu) or lower == "@" .. string.lower(fu) then
        return
    end

    local function esc(s)
        return (s:gsub("(%W)", "%%%1"))
    end

    -- match exacto DisplayName
    if rd ~= "" and lower == rd then
        obj.Text = fd
        return true
    end
    -- bloquear labels de racha/stats por nombre
    local on = obj.Name or ""
    local pn = obj.Parent and obj.Parent.Name or ""
    local meta = string.lower(on .. " " .. pn)
    if meta:find("streak", 1, true) or meta:find("racha", 1, true)
        or on == "Value" or on == "Wins" or on == "Kills" or on == "Level" or on == "Streak" then
        return false
    end
    -- match exacto Username → @fakeUser (NO el display name)
    if rn ~= "" and lower == rn then
        obj.Text = "@" .. tostring(fu):gsub("^@", "")
        return true
    end
    -- @username
    if rn ~= "" and lower == "@" .. rn then
        obj.Text = "@" .. tostring(fu):gsub("^@", "")
        return true
    end
    -- DisplayName @user juntos (evitar duplicar si ya tiene fake)
    if rd ~= "" and rn ~= "" and lower:find(rd, 1, true) and lower:find(rn, 1, true) then
        if lower:find(string.lower(fd), 1, true) then return false end
        obj.Text = fd .. "\n@" .. tostring(fu):gsub("^@", "")
        return true
    end
    -- contiene display name (leaderboard / lista) — una sola vez
    if rd ~= "" and #rd >= 2 and lower:find(rd, 1, true) and not lower:find(string.lower(fd), 1, true) then
        local ok, newT = pcall(function()
            return t:gsub(esc(realDisplay), fd, 1)
        end)
        if ok and newT and newT ~= t then
            obj.Text = newT
            return true
        end
    end
    -- contiene username → reemplazar por @user, no por display
    if rn ~= "" and #rn >= 3 and lower:find(rn, 1, true) and not lower:find(string.lower(fu), 1, true) then
        local ok, newT = pcall(function()
            if t:find("@" .. realName, 1, true) then
                return t:gsub(esc("@" .. realName), "@" .. tostring(fu):gsub("^@", ""), 1)
            end
            return t:gsub(esc(realName), tostring(fu):gsub("^@", ""), 1)
        end)
        if ok and newT and newT ~= t then
            obj.Text = newT
            return true
        end
    end
    return false
end
function Spoof.patchGuiTree(root, realName, realDisplay, fakeDisplay, fakeUser)
    if not root then return end
    pcall(function()
        for _, v in ipairs(root:GetDescendants()) do
            Spoof.patchTextObject(v, realName, realDisplay, fakeDisplay, fakeUser)
        end
    end)
end
function Spoof.patchMyProfileUI()
    if not config.SpoofEnabled or not config.SpoofMyName then return end
    if not config.SpoofProfileUI then return end
    if Spoof._lastPatch and tick() - Spoof._lastPatch < 1.0 then return end
    Spoof._lastPatch = tick()
    local fakeDisp = tostring(config.SpoofMyNameValue or "")
    if fakeDisp == "" then return end
    local fakeUser = Spoof.getMyUser()
    local realName = LocalPlayer.Name
    local realDisp = LocalPlayer.DisplayName or realName
    -- si el juego ya te puso el display spoofeado en Player, seguir usando Name real para buscar
    if string.lower(realDisp) == string.lower(fakeDisp) then
        realDisp = realName
    end
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if pg then Spoof.patchGuiTree(pg, realName, realDisp, fakeDisp, fakeUser) end
    -- OPT: NO escanear CoreGui completo (lag fuerte)
    -- billboards local desactivado (nametag propio = lag / doble nombre)
end
function Spoof.forceLocalDisplay()
    if not config.SpoofEnabled or not config.SpoofMyName then return end
    local fake = tostring(config.SpoofMyNameValue or "")
    if fake == "" then return end
    pcall(function()
        pcall(function() LocalPlayer.DisplayName = fake end)
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.DisplayName = fake end
    end)
end
function Spoof.hookLabel(obj)
    if not obj or Spoof.tagged[obj] then return end
    if not (obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox")) then return end
    Spoof.tagged[obj] = true
    pcall(function()
        obj:GetPropertyChangedSignal("Text"):Connect(function()
            if not config.SpoofEnabled or not config.SpoofMyName or not config.SpoofProfileUI then return end
            task.defer(function()
                Spoof.patchTextObject(
                    obj,
                    LocalPlayer.Name,
                    LocalPlayer.DisplayName,
                    tostring(config.SpoofMyNameValue),
                    Spoof.getMyUser()
                )
            end)
        end)
    end)
    -- parche inmediato
    if config.SpoofEnabled and config.SpoofMyName and config.SpoofProfileUI then
        Spoof.patchTextObject(
            obj,
            LocalPlayer.Name,
            LocalPlayer.DisplayName,
            tostring(config.SpoofMyNameValue),
            Spoof.getMyUser()
        )
    end
end
-- loop rapido: leaderboard + lista + perfil
task.spawn(function()
    while true do
        task.wait(8) -- menos agresivo = menos lag
        if not config.SpoofEnabled then continue end
        if config.SpoofMyName then
            Spoof.forceLocalDisplay()
            Spoof.patchMyProfileUI()
            pcall(PatchProfileStats) -- level/streak en perfil
        end
        -- no aplicar nametags a todos cada ciclo (lag)
    end
end)

-- GC periódico: conexiones muertas + tablas que crecen
task.spawn(function()
    while true do
        task.wait(45)
        pcall(function()
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.name_spoof_conn then
                for obj, conn in pairs(st.name_spoof_conn) do
                    if not obj or not obj.Parent then
                        pcall(function() if conn then conn:Disconnect() end end)
                        st.name_spoof_conn[obj] = nil
                    end
                end
            end
            if st and st.thumb_spoof_conn then
                for obj, conn in pairs(st.thumb_spoof_conn) do
                    if not obj or not obj.Parent then
                        pcall(function() if type(conn) == "userdata" and conn.Disconnect then conn:Disconnect() end end)
                        st.thumb_spoof_conn[obj] = nil
                    end
                end
            end
            if Spoof and Spoof.tagged then
                local n = 0
                for obj in pairs(Spoof.tagged) do
                    if not obj or not obj.Parent then
                        Spoof.tagged[obj] = nil
                    else
                        n = n + 1
                        if n > 400 then Spoof.tagged[obj] = nil end
                    end
                end
            end
            -- limpia HP cache de jugadores que salieron
            if SFX and SFX.hp then
                for plr in pairs(SFX.hp) do
                    if typeof(plr) == "Instance" and not plr:IsDescendantOf(Players) then
                        SFX.hp[plr] = nil
                    end
                end
            end
            -- limpia cache de nombres
            if getgenv()._ShakoOtherNames and tick() - (getgenv()._ShakoOtherNamesAt or 0) > 60 then
                getgenv()._ShakoOtherNames = nil
            end
        end)
        pcall(function()
            -- OPT: GC manual causa stutter; no forzar cada ciclo
        end)
    end
end)
-- UI nueva + cambios de texto (leaderboard/lista)
pcall(function()
    local pg = LocalPlayer:WaitForChild("PlayerGui", 10)
    if pg then
        -- solo al añadir texto nuevo (no escanear todo al inicio = menos lag)
        pg.DescendantAdded:Connect(function(obj)
            if not config.SpoofEnabled or not config.SpoofMyName then return end
            if not (obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox")) then return end
            -- throttle anti-lag
            local now = tick()
            if now - (getgenv()._ShakoSpoofAddAt or 0) < 0.05 then return end
            getgenv()._ShakoSpoofAddAt = now
            task.defer(function()
                Spoof.patchTextObject(
                    obj,
                    LocalPlayer.Name,
                    LocalPlayer.DisplayName,
                    tostring(config.SpoofMyNameValue),
                    Spoof.getMyUser()
                )
                -- si es UI de perfil, reaplicar stats
                local pn = obj.Parent and obj.Parent.Name or ""
                if pn == "Level" or pn == "Streak" or obj.Name == "Value" then
                    pcall(PatchProfileStats)
                end
            end)
        end)
    end
end)
-- CoreGui scan DESACTIVADO (causaba lag)
-- leftover from old connect - keep patch on add
pcall(function()
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if pg then
        pg.DescendantAdded:Connect(function(obj)
            if not config.SpoofEnabled or not config.SpoofMyName or not config.SpoofProfileUI then return end
            task.defer(function()
                if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
                    Spoof.patchTextObject(
                        obj,
                        LocalPlayer.Name,
                        LocalPlayer.DisplayName,
                        tostring(config.SpoofMyNameValue),
                        Spoof.getMyUser()
                    )
                end
            end)
        end)
    end
end)
Players.PlayerRemoving:Connect(function(p) Spoof.map[p] = nil end)

-- ==================== SKINCHANGER ====================
-- ==================== SKIN SPOOFER (solo visual) ====================
-- Copia ropa + accesorios + cara + piel desde otro UserId a TU character local
local InsertService = game:GetService("InsertService")
local Skin = {
    lastId = nil,
    applying = false,
    lastClothes = {},
    lastColors = nil,
    lastAccIds = {},
    lastFace = 0,
    keepConn = nil,
    cachedDesc = nil,
    lastKeep = 0,
    lastApply = 0,
}

function Skin.resolveUserId()
    local id = tonumber(config.SkinTargetUserId)
    if id and id > 0 then return id end
    local name = tostring(config.SkinTargetName or ""):gsub("^%s+", ""):gsub("%s+$", "")
    if name == "" then return nil end
    local ok, uid = pcall(function() return Players:GetUserIdFromNameAsync(name) end)
    if ok and type(uid) == "number" and uid > 0 then
        config.SkinTargetUserId = uid
        return uid
    end
    ok, uid = pcall(function() return Players:GetUserIdFromNameAsync(name:gsub("%s+", "")) end)
    if ok and type(uid) == "number" and uid > 0 then
        config.SkinTargetUserId = uid
        return uid
    end
    return nil
end

function Skin.clearAccessories(char)
    if not char then return end
    for _, v in ipairs(char:GetChildren()) do
        if v:IsA("Accessory") or v:IsA("Hat") or v:IsA("Accoutrement") then
            pcall(function() v:Destroy() end)
        end
    end
end

function Skin.clearClothes(char)
    if not char then return end
    for _, cls in ipairs({"Shirt", "Pants", "ShirtGraphic"}) do
        local o = char:FindFirstChildOfClass(cls)
        while o do
            pcall(function() o:Destroy() end)
            o = char:FindFirstChildOfClass(cls)
        end
    end
end

function Skin.cloneInto(char, src)
    if not char or not src then return 0 end
    local n = 0
    -- BodyColors
    pcall(function()
        local bc = src:FindFirstChildOfClass("BodyColors")
        if bc then
            local old = char:FindFirstChildOfClass("BodyColors")
            if old then old:Destroy() end
            local c = bc:Clone()
            c.Parent = char
            n = n + 1
        end
    end)
    -- Shirt / Pants / TShirt
    for _, cls in ipairs({"Shirt", "Pants", "ShirtGraphic"}) do
        pcall(function()
            local s = src:FindFirstChildOfClass(cls)
            if s then
                local old = char:FindFirstChildOfClass(cls)
                if old then old:Destroy() end
                local c = s:Clone()
                c.Parent = char
                n = n + 1
                if cls == "Shirt" then
                    Skin.lastClothes.shirt = true
                elseif cls == "Pants" then
                    Skin.lastClothes.pants = true
                end
            end
        end)
    end
    -- Face decal
    pcall(function()
        local sh = src:FindFirstChild("Head")
        local th = char:FindFirstChild("Head")
        if not sh or not th then return end
        local face = sh:FindFirstChild("face") or sh:FindFirstChildOfClass("Decal")
        if not face then return end
        local old = th:FindFirstChild("face") or th:FindFirstChildOfClass("Decal")
        if old then old:Destroy() end
        local f = face:Clone()
        f.Name = "face"
        f.Parent = th
        n = n + 1
    end)
    -- Body part colors
    pcall(function()
        for _, part in ipairs(src:GetChildren()) do
            if part:IsA("BasePart") then
                local dest = char:FindFirstChild(part.Name)
                if dest and dest:IsA("BasePart") then
                    dest.Color = part.Color
                    dest.Material = part.Material
                end
            end
        end
    end)
    -- Accessories
    local hum = char:FindFirstChildOfClass("Humanoid")
    for _, acc in ipairs(src:GetChildren()) do
        if acc:IsA("Accessory") or acc:IsA("Hat") or acc:IsA("Accoutrement") then
            pcall(function()
                local c = acc:Clone()
                local ok = false
                if hum then
                    ok = pcall(function() hum:AddAccessory(c) end)
                end
                if not ok then
                    c.Parent = char
                end
                n = n + 1
            end)
        end
    end
    return n
end

function Skin.applyFromDescription(char, humanoid, desc)
    if not char or not desc then return end
    -- colors
    pcall(function()
        Skin.lastColors = {
            Head = desc.HeadColor, Torso = desc.TorsoColor,
            LeftArm = desc.LeftArmColor, RightArm = desc.RightArmColor,
            LeftLeg = desc.LeftLegColor, RightLeg = desc.RightLegColor,
        }
        local old = char:FindFirstChildOfClass("BodyColors")
        if old then old:Destroy() end
        local bc = Instance.new("BodyColors")
        bc.HeadColor3 = desc.HeadColor
        bc.TorsoColor3 = desc.TorsoColor
        bc.LeftArmColor3 = desc.LeftArmColor
        bc.RightArmColor3 = desc.RightArmColor
        bc.LeftLegColor3 = desc.LeftLegColor
        bc.RightLegColor3 = desc.RightLegColor
        bc.Parent = char
    end)
    -- clothes via templates
    Skin.clearClothes(char)
    local function put(cls, prop, id)
        id = tonumber(id)
        if not id or id <= 0 then return end
        local o = Instance.new(cls)
        local urls = {
            "rbxassetid://" .. id,
            "http://www.roblox.com/asset/?id=" .. id,
            "rbxthumb://type=Asset&id=" .. id .. "&w=420&h=420",
        }
        for _, u in ipairs(urls) do
            pcall(function() o[prop] = u end)
            if o[prop] and o[prop] ~= "" then break end
        end
        o.Parent = char
    end
    put("Shirt", "ShirtTemplate", desc.Shirt)
    put("Pants", "PantsTemplate", desc.Pants)
    put("ShirtGraphic", "Graphic", desc.GraphicTShirt or desc.TShirt)
    Skin.lastClothes = {
        shirt = tonumber(desc.Shirt) or 0,
        pants = tonumber(desc.Pants) or 0,
        tshirt = tonumber(desc.GraphicTShirt) or 0,
    }
    -- face
    local faceId = tonumber(desc.Face) or 0
    if faceId > 0 then
        local head = char:FindFirstChild("Head")
        if head then
            local face = head:FindFirstChild("face") or head:FindFirstChildOfClass("Decal")
            if not face then
                face = Instance.new("Decal")
                face.Name = "face"
                face.Face = Enum.NormalId.Front
                face.Parent = head
            end
            pcall(function() face.Texture = "rbxassetid://" .. faceId end)
            Skin.lastFace = faceId
        end
    end
    -- accessories from description strings
    local ids = {}
    for _, field in ipairs({
        "HatAccessory", "HairAccessory", "FaceAccessory", "NeckAccessory",
        "ShouldersAccessory", "FrontAccessory", "BackAccessory", "WaistAccessory",
    }) do
        local val = ""
        pcall(function() val = tostring(desc[field] or "") end)
        for id in string.gmatch(val, "%d+") do
            table.insert(ids, tonumber(id))
        end
    end
    pcall(function()
        for _, a in ipairs(desc:GetAccessories(true) or {}) do
            local id = tonumber(a.AssetId)
            if id then table.insert(ids, id) end
        end
    end)
    Skin.lastAccIds = ids
    Skin.clearAccessories(char)
    for _, id in ipairs(ids) do
        pcall(function()
            local model = InsertService:LoadAsset(id)
            if not model then return end
            local acc = model:FindFirstChildOfClass("Accessory")
                or model:FindFirstChildOfClass("Hat")
                or model:FindFirstChildWhichIsA("Accoutrement", true)
            if not acc then
                for _, d in ipairs(model:GetDescendants()) do
                    if d:IsA("Accessory") or d:IsA("Hat") then acc = d break end
                end
            end
            if acc then
                local c = acc:Clone()
                local ok = humanoid and pcall(function() humanoid:AddAccessory(c) end)
                if not ok then c.Parent = char end
            end
            model:Destroy()
        end)
        task.wait()
    end
end

function Skin.reapplyColors()
    local char = LocalPlayer.Character
    if not char or not Skin.lastColors then return end
    local c = Skin.lastColors
    local map = {
        Head = c.Head, Torso = c.Torso,
        ["Left Arm"] = c.LeftArm, ["Right Arm"] = c.RightArm,
        ["Left Leg"] = c.LeftLeg, ["Right Leg"] = c.RightLeg,
        UpperTorso = c.Torso, LowerTorso = c.Torso,
        LeftUpperArm = c.LeftArm, LeftLowerArm = c.LeftArm, LeftHand = c.LeftArm,
        RightUpperArm = c.RightArm, RightLowerArm = c.RightArm, RightHand = c.RightArm,
        LeftUpperLeg = c.LeftLeg, LeftLowerLeg = c.LeftLeg, LeftFoot = c.LeftLeg,
        RightUpperLeg = c.RightLeg, RightLowerLeg = c.RightLeg, RightFoot = c.RightLeg,
    }
    for name, col in pairs(map) do
        local part = char:FindFirstChild(name)
        if part and part:IsA("BasePart") and col then
            pcall(function() part.Color = col end)
        end
    end
end

function Skin.copyAvatar(userId)
    userId = tonumber(userId)
    if not userId or userId <= 0 then
        Notify("Skin", "UserId invalido")
        return false
    end
    if Skin.applying then
        Notify("Skin", "Espera...")
        return false
    end
    Skin.applying = true
    Skin.lastId = userId
    config.SkinTargetUserId = userId
    config.SkinchangerEnabled = true

    local char = LocalPlayer.Character
    if not char then
        Skin.applying = false
        Notify("Skin", "Sin character")
        return false
    end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        Skin.applying = false
        return false
    end

    Notify("Skin", "Cargando " .. tostring(userId) .. "...")

    local copied = 0
    local method = "none"

    -- METODO 1 (mejor): modelo completo del avatar
    local model
    local okm, errm = pcall(function()
        model = Players:CreateHumanoidModelFromUserIdAsync(userId)
    end)
    if okm and model then
        method = "Model"
        copied = Skin.cloneInto(char, model)
        pcall(function() model:Destroy() end)
    else
        warn("[Skin] Model fail", errm)
    end

    -- METODO 2: HumanoidDescription
    local desc
    pcall(function()
        desc = Players:GetHumanoidDescriptionFromUserIdAsync(userId)
    end)
    if desc then
        -- try official apply
        pcall(function()
            if humanoid.ApplyDescriptionReset then
                humanoid:ApplyDescriptionReset(desc)
            elseif humanoid.ApplyDescriptionAsync then
                humanoid:ApplyDescriptionAsync(desc)
            else
                humanoid:ApplyDescription(desc)
            end
            method = method .. "+ApplyDesc"
        end)
        task.wait(0.2)
        char = LocalPlayer.Character or char
        humanoid = char:FindFirstChildOfClass("Humanoid") or humanoid
        -- force clothes/acc again
        Skin.applyFromDescription(char, humanoid, desc)
        method = method .. "+Desc"
        copied = copied + 1
    end

    -- METODO 3: AppearanceInfo
    pcall(function()
        local info = Players:GetCharacterAppearanceInfoAsync(userId)
        if type(info) == "table" and info.assets then
            for _, asset in ipairs(info.assets) do
                local id = tonumber(asset.id)
                if id then
                    pcall(function()
                        local m = InsertService:LoadAsset(id)
                        if not m then return end
                        local shirt = m:FindFirstChildOfClass("Shirt")
                        local pants = m:FindFirstChildOfClass("Pants")
                        local acc = m:FindFirstChildOfClass("Accessory") or m:FindFirstChildOfClass("Hat")
                        if shirt then
                            Skin.clearClothes(char)
                            shirt:Clone().Parent = char
                        end
                        if pants then
                            pants:Clone().Parent = char
                        end
                        if acc then
                            local c = acc:Clone()
                            if not pcall(function() humanoid:AddAccessory(c) end) then
                                c.Parent = char
                            end
                        end
                        m:Destroy()
                    end)
                end
            end
            method = method .. "+Appearance"
        end
    end)

    -- guardar desc para reaplicar siempre
    if desc then Skin.cachedDesc = desc end
    Skin.lastApply = tick()

    -- KEEP PERMANENTE: Rivals borra ropa a los ~1s, reponer mientras Skin ON
    if Skin.keepConn then pcall(function() Skin.keepConn:Disconnect() end) Skin.keepConn = nil end
    if Skin.keepThread then pcall(function() task.cancel(Skin.keepThread) end) Skin.keepThread = nil end
    Skin.keepThread = task.spawn(function()
        while config.SkinchangerEnabled or Skin.lastId do
            task.wait(1.5)
            if Skin.applying then continue end
            local c = LocalPlayer.Character
            if not c then continue end
            local hum = c:FindFirstChildOfClass("Humanoid")
            local needClothes = not c:FindFirstChildOfClass("Shirt") or not c:FindFirstChildOfClass("Pants")
            local accN = 0
            for _, v in ipairs(c:GetChildren()) do
                if v:IsA("Accessory") or v:IsA("Hat") then accN = accN + 1 end
            end
            local needAcc = accN < 2  -- reponer si Rivals limpia casi todo
            if needClothes or needAcc then
                if Skin.cachedDesc then
                    pcall(function() Skin.applyFromDescription(c, hum, Skin.cachedDesc) end)
                end
                if Skin.lastId then
                    pcall(function() Skin.copyAvatar(Skin.lastId) end)
                end
            end
            pcall(Skin.reapplyColors)
        end
        Skin.keepThread = nil
    end)

    Skin.applying = false
    local hasShirt = char:FindFirstChildOfClass("Shirt") ~= nil
    local hasPants = char:FindFirstChildOfClass("Pants") ~= nil
    local acc = 0
    for _, v in ipairs(char:GetChildren()) do
        if v:IsA("Accessory") or v:IsA("Hat") then acc = acc + 1 end
    end
    Notify("Skin", string.format("%s | shirt:%s pants:%s acc:%d", method, tostring(hasShirt), tostring(hasPants), acc))
    if not hasShirt and not hasPants and acc == 0 then
        Notify("Skin", "Nada se aplico — UserId mal o executor bloquea assets")
    end
    return hasShirt or hasPants or acc > 0
end

function Skin.SpoofSkin(userId)
    return Skin.copyAvatar(userId)
end

function Skin.apply()
    local id = Skin.resolveUserId()
    if not id then
        Notify("Skin", "Pon UserId o nombre Roblox")
        return false
    end
    return Skin.copyAvatar(id)
end

LocalPlayer.CharacterAdded:Connect(function()
    if not (config.SkinchangerEnabled or Skin.lastId) then return end
    local id = Skin.lastId or tonumber(config.SkinTargetUserId)
    if not id or id <= 0 then return end
    if not config.SkinOnRespawn and not config.SkinchangerEnabled then return end
    task.spawn(function()
        for _, d in ipairs({0.5, 1.2, 2.0, 3.5, 5.0}) do
            task.wait(d)
            if not (config.SkinchangerEnabled or Skin.lastId) then return end
            if LocalPlayer.Character then
                pcall(function() Skin.copyAvatar(id) end)
            end
        end
    end)
end)

task.spawn(function()
    while true do
        task.wait(1)
        if (config.SkinchangerEnabled or Skin.lastId) and LocalPlayer.Character then
            Skin.reapplyColors()
        end
    end
end)





function StartTriggerbot()
    SafeDisconnect("Triggerbot")
    local tbFrame = 0
    Connections.Triggerbot = RunService.RenderStepped:Connect(function()
        tbFrame += 1
        if tbFrame % 2 ~= 0 then return end -- OPT
        if not config.TriggerbotEnabled or UserInputService:GetFocusedTextBox() then return end
        local partName = config.TriggerbotPart or "Head"
        local mousePos = UserInputService:GetMouseLocation()
        local hit = false
        for _, plr in pairs(Players:GetPlayers()) do
            if plr == LocalPlayer or not plr.Character then continue end
            if config.TeamCheck and IsSameTeam(plr) then continue end
            local part = plr.Character:FindFirstChild(partName) or plr.Character:FindFirstChild("Head")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if not part or not hum or hum.Health <= 0 then continue end
            local sp, on = Camera:WorldToViewportPoint(part.Position)
            if on and sp.Z > 0 and (Vector2.new(sp.X, sp.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude <= (config.TriggerbotFOV or 40) then
                hit = true
                break
            end
        end
        if hit and tick() - lastTriggerClick >= 1 / math.max(config.TriggerbotCPS or 12, 1) then
            lastTriggerClick = tick()
            pcall(function() mouse1click() end)
        end
    end)
end
function StopTriggerbot() SafeDisconnect("Triggerbot") end

-- Rapid Fire


local StartSilentAim, StopSilentAim, StartFOVDraw, silentTarget, silentCFD, loadSilentDeps, updateFOVCircle, fovColor, isVisibleWall
function InitCombatHooks()
-- ==================== FOV estilo Grief.cc (ScreenGui + gradient + spin) ====================
local FOV_COLORS = {
    White = Color3.fromRGB(255, 255, 255),
    Red = Color3.fromRGB(255, 70, 80),
    Green = Color3.fromRGB(80, 255, 140),
    Cyan = Color3.fromRGB(80, 220, 255),
    Magenta = Color3.fromRGB(255, 100, 220),
    Yellow = Color3.fromRGB(255, 230, 80),
    Orange = Color3.fromRGB(255, 160, 50),
    Purple = Color3.fromRGB(168, 85, 255),
}
fovColor = function(name)
    if typeof(name) == "Color3" then return name end
    return FOV_COLORS[name or "White"] or Color3.fromRGB(255, 255, 255)
end

local fovScreenGui
local function ensureFovGui()
    if fovScreenGui and fovScreenGui.Parent then return fovScreenGui end
    local parent
    pcall(function() parent = gethui and gethui() end)
    if not parent then pcall(function() parent = game:GetService("CoreGui") end) end
    if not parent then parent = LocalPlayer:FindFirstChild("PlayerGui") end
    local g = Instance.new("ScreenGui")
    g.Name = "ShakoFOV_Grief"
    g.DisplayOrder = 99980
    g.ResetOnSpawn = false
    g.IgnoreGuiInset = true
    g.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    g.Parent = parent
    fovScreenGui = g
    return g
end

local function buildGriefFOV(name, cfg)
    local gui = ensureFovGui()
    local container = Instance.new("Frame")
    container.Name = name
    container.BackgroundTransparency = 1
    container.BorderSizePixel = 0
    container.Visible = false
    container.AnchorPoint = Vector2.new(0.5, 0.5)
    container.Parent = gui

    local fill = Instance.new("Frame")
    fill.Size = UDim2.fromScale(1, 1)
    fill.BackgroundColor3 = Color3.new(1, 1, 1)
    fill.BackgroundTransparency = cfg.fillTrans or 0.92
    fill.BorderSizePixel = 0
    fill.ZIndex = 1
    fill.Parent = container
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
    local fillgrad = Instance.new("UIGradient")
    fillgrad.Color = ColorSequence.new(cfg.col1 or Color3.new(1,1,1), cfg.col2 or Color3.new(1,1,1))
    fillgrad.Parent = fill

    local outline = Instance.new("Frame")
    outline.Size = UDim2.fromScale(1, 1)
    outline.BackgroundTransparency = 1
    outline.BorderSizePixel = 0
    outline.ZIndex = 2
    outline.Parent = container
    Instance.new("UICorner", outline).CornerRadius = UDim.new(1, 0)
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = cfg.thickness or 1.5
    stroke.Transparency = cfg.outlineTrans or 0.08
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    stroke.Parent = outline
    local strokegrad = Instance.new("UIGradient")
    strokegrad.Color = ColorSequence.new(cfg.col1 or Color3.new(1,1,1), cfg.col2 or Color3.fromRGB(180,180,255))
    strokegrad.Parent = stroke

    return { container = container, fill = fill, fillgrad = fillgrad, stroke = stroke, strokegrad = strokegrad }
end

local silentFOVUI = buildGriefFOV("SilentFOV", {
    fillTrans = 0.92, thickness = 1.5,
    col1 = Color3.fromRGB(255,255,255), col2 = Color3.fromRGB(180,200,255),
})
local aimbotFOVUI = buildGriefFOV("AimbotFOV", {
    fillTrans = 0.94, thickness = 1.5,
    col1 = Color3.fromRGB(255,210,80), col2 = Color3.fromRGB(255,120,50),
})

local function applyFOVUI(ui, enabled, show, radius, colName, doFill, fillTrans, animated, spin)
    if not ui or not ui.container then return end
    local cam = workspace.CurrentCamera
    if not cam then return end
    local vis = enabled and show
    ui.container.Visible = vis and true or false
    if not vis then return end
    local col = fovColor(colName)
    local col2 = Color3.new(math.clamp(col.R*0.65+0.2,0,1), math.clamp(col.G*0.65+0.2,0,1), math.clamp(col.B*0.8+0.25,0,1))
    local r = tonumber(radius) or 120
    if animated then r = r * (0.97 + 0.03 * math.sin(tick() * 2.2)) end
    local center = cam.ViewportSize / 2
    ui.container.Size = UDim2.fromOffset(r * 2, r * 2)
    ui.container.Position = UDim2.fromOffset(center.X, center.Y)
    if ui.fill then
        ui.fill.Visible = doFill ~= false
        ui.fill.BackgroundTransparency = math.clamp(tonumber(fillTrans) or 0.92, 0.5, 0.98)
    end
    local spd = tonumber(config.SilentFOVSpinSpeed) or 1.2
    if ui.fillgrad then
        ui.fillgrad.Color = ColorSequence.new(col, col2)
        if animated or spin then ui.fillgrad.Rotation = (tick() * spd * 90) % 360 end
    end
    if ui.strokegrad then
        ui.strokegrad.Color = ColorSequence.new(col, col2)
        if spin or animated then ui.strokegrad.Rotation = (tick() * spd * 90 + 90) % 360 end
    end
    if ui.stroke then
        ui.stroke.Thickness = tonumber(config.SilentFOVThickness) or 1.5
    end
end

local SilentFOVOutline, SilentFOVFill, SilentFOVGlow
local AimbotFOVOutline, AimbotFOVFill, AimbotFOVGlow
updateFOVCircle = function() end

Connections.FOVDraw = nil
StartFOVDraw = function()
    SafeDisconnect("FOVDraw")
    local fovFrame = 0
    Connections.FOVDraw = RunService.Heartbeat:Connect(function()
        -- skip si ninguno visible
        local sOn = config.SilentAimEnabled and config.SilentShowFOV ~= false
        local aOn = config.AimbotEnabled and config.AimbotShowFOV ~= false
        if not sOn and not aOn then
            if silentFOVUI and silentFOVUI.container then silentFOVUI.container.Visible = false end
            if aimbotFOVUI and aimbotFOVUI.container then aimbotFOVUI.container.Visible = false end
            return
        end
        fovFrame = fovFrame + 1
        -- anim/spin: cada frame; si no, cada 2
        local needFast = (sOn and (config.SilentFOVAnimated or config.SilentFOVSpin)) or (aOn and (config.AimbotFOVAnimated or config.AimbotFOVSpin))
        if not needFast and fovFrame % 2 == 0 then return end
        if sOn then
            applyFOVUI(silentFOVUI, true, true,
                config.SilentFOV or 300, config.SilentFOVColor or "White",
                config.SilentFOVFill ~= false, config.SilentFOVFillTransparency,
                config.SilentFOVAnimated, config.SilentFOVSpin ~= false)
        else
            if silentFOVUI and silentFOVUI.container then silentFOVUI.container.Visible = false end
        end
        if aOn then
            applyFOVUI(aimbotFOVUI, true, true,
                config.AimbotFOV or 180, config.AimbotFOVColor or "Yellow",
                config.AimbotFOVFill ~= false, config.AimbotFOVFillTransparency,
                config.AimbotFOVAnimated, config.AimbotFOVSpin == true)
        else
            if aimbotFOVUI and aimbotFOVUI.container then aimbotFOVUI.container.Visible = false end
        end
    end)
end
StartFOVDraw()

isVisibleWall = function(fromPos, toPos, targetChar)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local filter = { LocalPlayer.Character }
    if targetChar then table.insert(filter, targetChar) end
    params.FilterDescendantsInstances = filter
    params.IgnoreWater = true
    local dir = toPos - fromPos
    local res = workspace:Raycast(fromPos, dir, params)
    return res == nil
end


-- ==================== SILENT AIM v3 (Rivals — head · no rompe balas/ammo) ====================
local GameplayUtility, GunModule
local silentHooked = false
local silentOldFire, silentOldRaycast, silentOldNamecall

loadSilentDeps = function()
    pcall(function()
        GameplayUtility = require(RS.Modules.GameplayUtility)
    end)
    pcall(function()
        local ok, gun = pcall(require, LocalPlayer.PlayerScripts.Modules.ItemTypes.Gun)
        if ok and gun then GunModule = gun end
    end)
    if not util or not enum then
        pcall(loadRivalsModules)
    end
end
task.spawn(loadSilentDeps)

-- Solo Head en FOV
silentTarget = function()
    if not config.SilentAimEnabled then return nil end
    local cam = workspace.CurrentCamera
    if not cam then return nil end
    local center = cam.ViewportSize / 2
    local fov = math.max(tonumber(config.SilentFOV) or 2000, 50)
    local bestHead, bestDist = nil, fov
    local myChar = LocalPlayer.Character

    local function tryHead(char)
        if not char or char == myChar then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum or hum.Health <= 0 then return end
        local head = char:FindFirstChild("Head")
        if not head then return end
        local sp, onScreen = cam:WorldToViewportPoint(head.Position)
        if sp.Z <= 0 then return end
        local d = (Vector2.new(sp.X, sp.Y) - Vector2.new(center.X, center.Y)).Magnitude
        if not onScreen and d > fov then return end
        if d < bestDist then bestDist, bestHead = d, head end
    end

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if config.SilentTeamCheck and IsEnemy and not IsEnemy(plr) then continue end
        tryHead(plr.Character)
    end
    pcall(function()
        for _, entity in ipairs(game:GetService("CollectionService"):GetTagged("Entity")) do
            local plr = Players:GetPlayerFromCharacter(entity)
            if plr == LocalPlayer then continue end
            if plr and config.SilentTeamCheck and IsEnemy and not IsEnemy(plr) then continue end
            tryHead(entity)
        end
    end)
    return bestHead
end

-- Prediccion suave: solo horizontal, sin over-lead
local function silentPredictPos(head, origin)
    local pos = head.Position
    if config.SilentPredict == false then return pos end
    local char = head.Parent
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return pos end
    local vel = Vector3.zero
    pcall(function() vel = root.AssemblyLinearVelocity end)
    -- ignorar micro movimiento / ruido
    if vel.Magnitude < 8 then return pos end
    -- solo horizontal (Y del salto no leadear tanto)
    local vFlat = Vector3.new(vel.X, vel.Y * 0.25, vel.Z)
    local t = math.clamp(tonumber(config.SilentPredictAmount) or 0.06, 0, 0.25)
    if origin then
        local dist = (pos - origin).Magnitude
        -- lead corto segun distancia (no full flight time)
        t = t + math.clamp(dist / 2500, 0, 0.05)
    end
    return pos + vFlat * t
end

silentCFD = function(origin, head)
    if not util or not head or not origin then return nil end
    local targetPos = silentPredictPos(head, origin)
    -- un poco ARRIBA de la head (evita pecho)
    local up = tonumber(config.SilentHeadOffsetY) or 0.35
    targetPos = targetPos + Vector3.new(0, up, 0)
    local aimCF = CFrame.lookAt(origin, targetPos)
    local targetCF = head.CFrame
    local d = {}
    d[utf8.char(1)] = {
        [utf8.char(0)] = util:EncodeCFrame(aimCF),
        [utf8.char(1)] = util:EncodeCFrame(targetCF),
        [utf8.char(2)] = head,
        [utf8.char(3)] = util:EncodeCFrame(targetCF:ToObjectSpace(CFrame.new(targetPos))),
    }
    return d
end

-- SOLO StartShooting real (no numbers, no enums random → no rompe ammo)
local function silentIsStartShooting(enumVal)
    if enumVal == nil or not enum then return false end
    local ok, shoot = pcall(function() return enum:ToEnum("StartShooting") end)
    if ok and shoot ~= nil and enumVal == shoot then return true end
    return false
end

StartSilentAim = function()
    loadSilentDeps()
    if not util or not enum then
        pcall(loadRivalsModules)
        loadSilentDeps()
    end
    if not util or not enum then
        Notify("Silent", "Modules no listos — reintenta")
        return
    end

    -- Force ADS helper (sniper) — no tocar cooldowns ni ammo
    pcall(function()
        local ok, gun = pcall(require, LocalPlayer.PlayerScripts.Modules.ItemTypes.Gun)
        if ok and gun and gun.IsFullyAiming then
            gun.IsFullyAiming = function() return true end
        end
    end)

    local function patchCamdata(camdata)
        local t = silentTarget()
        if not t or not util then return camdata end -- sin target: NO tocar (ammo/hits normales)
        local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not root then return camdata end
        local built = silentCFD(root.Position, t)
        if not built then return camdata end
        -- merge: conservar el resto del packet, solo reemplazar entry de aim
        if type(camdata) ~= "table" then
            return built
        end
        local out = {}
        for k, v in pairs(camdata) do out[k] = v end
        for k, v in pairs(built) do out[k] = v end
        return out
    end

    -- Raycast: redirigir direccion SOLO si hay head target (no tocar maxDist agresivo)
    pcall(function()
        if silentOldRaycast then return end
        if not GameplayUtility then
            pcall(function() GameplayUtility = require(RS.Modules.GameplayUtility) end)
        end
        if GameplayUtility and GameplayUtility.GetEntitiesFromRaycast then
            silentOldRaycast = GameplayUtility.GetEntitiesFromRaycast
            GameplayUtility.GetEntitiesFromRaycast = function(self, envID, params, origin, dir, maxDist, ...)
                if config.SilentAimEnabled then
                    local t = silentTarget()
                    if t and origin and typeof(origin) == "Vector3" then
                        local aimPos = silentPredictPos(t, origin) + Vector3.new(0, tonumber(config.SilentHeadOffsetY) or 0.35, 0)
                        local to = aimPos - origin
                        if to.Magnitude > 0.05 then
                            dir = to.Unit
                            if type(maxDist) == "number" and to.Magnitude > maxDist then
                                maxDist = to.Magnitude + 2
                            end
                        end
                    end
                end
                return silentOldRaycast(self, envID, params, origin, dir, maxDist, ...)
            end
        end
    end)

    local UseItem
    pcall(function()
        UseItem = RS.Remotes.Replication.Fighter.UseItem
    end)

    -- Un solo hook limpio: solo StartShooting + solo si hay target
    if UseItem and hookfunction and not silentOldFire then
        pcall(function()
            silentOldFire = hookfunction(UseItem.FireServer, newcclosure(function(self, objID, enumVal, camdata, extra, ...)
                if config.SilentAimEnabled and silentIsStartShooting(enumVal) then
                    local patched = patchCamdata(camdata)
                    -- si patch fallo, mandar original (no romper tiro)
                    if patched ~= nil then camdata = patched end
                end
                -- SIEMPRE llamar original → ammo + hits se procesan normal
                return silentOldFire(self, objID, enumVal, camdata, extra, ...)
            end))
        end)
    end

    if not silentOldFire and not silentOldNamecall and hookmetamethod then
        pcall(function()
            silentOldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                local method = getnamecallmethod and getnamecallmethod() or ""
                if config.SilentAimEnabled and method == "FireServer" then
                    local UseItemRemote
                    pcall(function() UseItemRemote = RS.Remotes.Replication.Fighter.UseItem end)
                    if self == UseItemRemote then
                        local n = select("#", ...)
                        local args = {...}
                        if silentIsStartShooting(args[2]) then
                            args[3] = patchCamdata(args[3])
                            return silentOldNamecall(self, table.unpack(args, 1, n))
                        end
                    end
                end
                return silentOldNamecall(self, ...)
            end))
        end)
    end

    silentHooked = true
    config.SilentAimEnabled = true
    config.SilentHitPart = "Head"
    Notify("Silent", "ON · head · ammo OK")
end

StopSilentAim = function()
    config.SilentAimEnabled = false
    Notify("Silent", "OFF")
end

end -- InitCombatHooks

InitCombatHooks()

function StartRapidFire()
    config.RapidFire = true
    if ApplyItemLibraryGuns(true) then
        Notify("ItemLibrary", "Fast Fire + NoRecoil props ON")
    else
        Notify("ItemLibrary", "FAIL — Modules.ItemLibrary")
    end
end
function StopRapidFire()
    config.RapidFire = false
    ApplyItemLibraryGuns(false)
    if config.FastMelee then ApplyItemLibraryMelee(true) end
    Notify("ItemLibrary", "Fast Fire OFF (restored)")
end
function StartFastMelee()
    config.FastMelee = true
    ApplyItemLibraryMelee(true)
    Notify("ItemLibrary", "Fast Melee ON")
end
function StopFastMelee()
    config.FastMelee = false
    ApplyItemLibraryMelee(false)
    if config.RapidFire then ApplyItemLibraryGuns(true) end
    Notify("ItemLibrary", "Fast Melee OFF")
end

-- No Recoil scan

-- Targeting
function GetTargetByName()
    if not config.UseNameTarget or config.TargetName == "" then return nil end
    local n = string.lower(config.TargetName)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and string.lower(plr.Name):find(n) then
            local hum = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 and IsEnemy(plr) then return plr end
        end
    end
    return nil
end
function ScoreTarget(plr, myHRP)
    local root = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
    local head = plr.Character and plr.Character:FindFirstChild("Head")
    local hum = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
    if not root or not hum or hum.Health <= 0 then return math.huge end
    local dist = (myHRP.Position - root.Position).Magnitude
    if not config.InfiniteRange and not config.RageInfinite then
        local mr = tonumber(config.MaxRange) or 1e9
        if dist > mr then return math.huge end
    end
    if config.PriorityMode == "LowestHP" then return hum.Health + dist * 0.05 end
    if config.PriorityMode == "Crosshair" and head then
        local sp, on = Camera:WorldToViewportPoint(head.Position)
        if not on then return math.huge end
        local m = UserInputService:GetMouseLocation()
        return (Vector2.new(sp.X, sp.Y) - Vector2.new(m.X, m.Y)).Magnitude
    end
    return dist
end
function GetClosestEnemy()
    if config.TargetLock and LockedTarget and LockedTarget.Parent and LockedTarget.Character then
        local hum = LockedTarget.Character:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health > 0 and IsEnemy(LockedTarget) then
            return LockedTarget
        else LockedTarget = nil end
    end
    local named = GetTargetByName()
    if named and IsEnemy(named) then
        if config.TargetLock then LockedTarget = named end
        return named
    end
    local myHRP = getHRP()
    if not myHRP then return nil end
    local best, bestScore = nil, math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == LocalPlayer or not plr.Character then continue end
        if not IsEnemy(plr) then continue end -- nunca team
        local score = ScoreTarget(plr, myHRP)
        if score < bestScore then bestScore, best = score, plr end
    end
    if best and config.TargetLock then LockedTarget = best end
    return best
end
function PredictPos(root, head)
    if not config.RagebotPredict or not root then return head and head.Position or root.Position end
    return (head and head.Position or root.Position) + root.AssemblyLinearVelocity * (config.RagebotPredictAmount or 0.12)
end
local _rageWeakUntil = _rageWeakUntil or {}

function GetRagebotTarget()
    local myHRP = getHRP()
    if not myHRP then return nil end

    local function validRage(plr)
        if not plr or not plr.Character then return false end
        if not isEnemyRageRivals(plr) then return false end
        local hum = plr.Character:FindFirstChildOfClass("Humanoid")
        local head = plr.Character:FindFirstChild("Head")
        if not hum or not head or hum.Health <= 0 then return false end
        if config.RagebotSkipFF and plr.Character:FindFirstChildOfClass("ForceField") then return false end
        return true
    end

    local function ragerScore(plr)
        local char = plr.Character
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        local head = char:FindFirstChild("Head")
        if not hum or not root or not head then return 0 end
        local bonus = 0
        if hum.Health < 40 then bonus = bonus + 80 end
        if hum.Health < 20 then bonus = bonus + 60 end
        local ff = char:FindFirstChildOfClass("ForceField")
        if not ff then
            bonus = bonus + 40
            if _rageWeakUntil[plr] and tick() < _rageWeakUntil[plr] then
                bonus = bonus + 120
            end
        else
            _rageWeakUntil[plr] = tick() + 1.2
        end
        local vel = root.AssemblyLinearVelocity.Magnitude
        if vel > 80 then bonus = bonus + 50 end
        if vel > 150 then bonus = bonus + 70 end
        if type(hasKnifeViewModel) == "function" and hasKnifeViewModel(plr) then
            bonus = bonus + 90
        end
        local d = (myHRP.Position - head.Position).Magnitude
        if d < 25 then bonus = bonus + 55 end
        if d < 12 then bonus = bonus + 40 end
        if config.RagePrioritizeRagers == false then bonus = bonus * 0.25 end
        return bonus
    end

    if config.TargetLock and LockedTarget and LockedTarget.Parent and validRage(LockedTarget) then
        return LockedTarget
    end
    if RagebotTarget and RagebotTarget.Parent and validRage(RagebotTarget) then
        local head = RagebotTarget.Character:FindFirstChild("Head")
        if head and (config.RagebotInfiniteRange or config.RagebotRange == math.huge
            or (myHRP.Position - head.Position).Magnitude <= (config.RagebotRange or 1e12) * 1.5) then
            local curB = ragerScore(RagebotTarget)
            local better = false
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= LocalPlayer and plr ~= RagebotTarget and validRage(plr) then
                    if ragerScore(plr) > curB + 80 then better = true break end
                end
            end
            if not better then return RagebotTarget end
        end
    end

    RagebotTarget = nil
    local best, bestScore = nil, math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == LocalPlayer or not plr.Character then continue end
        if not validRage(plr) then continue end
        local hum = plr.Character:FindFirstChildOfClass("Humanoid")
        local head = plr.Character:FindFirstChild("Head")
        local d = (myHRP.Position - head.Position).Magnitude
        if not config.RagebotInfiniteRange and config.RagebotRange ~= math.huge and d > (config.RagebotRange or 1e12) then continue end
        local score = (config.RagebotPriorityHP and (hum.Health + d * 0.1) or d) - ragerScore(plr)
        if score < bestScore then bestScore, best = score, plr end
    end
    if best then
        if RagebotTarget and RagebotTarget ~= best then
            local delay = tonumber(config.RageSwitchDelay) or 0.05
            if tick() - (LastRageSwitch or 0) < delay then
                if validRage(RagebotTarget) then return RagebotTarget end
            end
        end
        LastRageSwitch = tick()
        if config.TargetLock then LockedTarget = best end
        RagebotTarget = best
    end
    return best
end

function AutoEquipMelee()
    -- desactivado (pedido del user)
end
function DoAutoEquip() end
function StartAutoEquip() end
function StopAutoEquip() end

function ApplyFakePosition()
    if not config.FakePosition then return end
    local myHRP = getHRP()
    if not myHRP then return end
    if not FakePart or not FakePart.Parent then
        FakePart = Instance.new("Part")
        FakePart.Name = "FakePos_Expert"
        FakePart.Size = Vector3.new(2, 2, 1)
        FakePart.Transparency = 1
        FakePart.CanCollide = false
        FakePart.Anchored = true
        FakePart.Parent = workspace
    end
    local s = config.FakePosStrength
    FakePart.CFrame = myHRP.CFrame * CFrame.new(math.random(-s, s), math.random(-s / 2, s / 2), math.random(-s, s))
end
function AntiSafeZone()
    -- DESACTIVADO: ya no te manda al cielo
    return
end
function isSlideKeyDown()
    local key = tostring(config.SlideBoostKey or "C"):upper()
    -- C solo = C (NO Control)
    if key == "C" then
        return UserInputService:IsKeyDown(Enum.KeyCode.C)
    end
    if key == "LEFTCONTROL" or key == "LEFTCTRL" or key == "CTRL" then
        return UserInputService:IsKeyDown(Enum.KeyCode.LeftControl)
            or UserInputService:IsKeyDown(Enum.KeyCode.RightControl)
    end
    if key == "LEFTSHIFT" or key == "SHIFT" then
        return UserInputService:IsKeyDown(Enum.KeyCode.LeftShift)
            or UserInputService:IsKeyDown(Enum.KeyCode.RightShift)
    end
    local ok, kc = pcall(function() return Enum.KeyCode[key] end)
    if ok and kc then return UserInputService:IsKeyDown(kc) end
    -- default seguro: solo C
    return UserInputService:IsKeyDown(Enum.KeyCode.C)
end

function getMoveDir()
    local hum = getHum()
    if hum and hum.MoveDirection.Magnitude > 0.05 then
        return hum.MoveDirection
    end
    -- fallback WASD por camara
    local cam = workspace.CurrentCamera
    if not cam then return Vector3.zero end
    local cf = cam.CFrame
    local f = Vector3.new(cf.LookVector.X, 0, cf.LookVector.Z)
    local r = Vector3.new(cf.RightVector.X, 0, cf.RightVector.Z)
    if f.Magnitude > 0 then f = f.Unit end
    if r.Magnitude > 0 then r = r.Unit end
    local dir = Vector3.zero
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + f end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - f end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + r end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - r end
    if dir.Magnitude > 0 then return dir.Unit end
    return Vector3.zero
end

function ApplySpeedState()
    local hum = getHum()
    if not hum then return end
    if config.SpeedEnabled then
        local spd = config.WalkSpeed or 50
        pcall(function() hum.WalkSpeed = spd end)
    else
        -- restaurar default (no dejar speed activo)
        local d = config.DefaultWalkSpeed or 16
        pcall(function()
            if hum.WalkSpeed ~= d then
                hum.WalkSpeed = d
            end
        end)
    end
end

function ApplyJumpState()
    local hum = getHum()
    if not hum then return end
    if config.JumpPowerEnabled then
        local jp = config.JumpPower or 100
        pcall(function()
            hum.UseJumpPower = true
            hum.JumpPower = jp
            if hum.JumpHeight ~= nil then
                hum.JumpHeight = math.clamp(jp / 20, 2, 500)
            end
        end)
    else
        local dj = config.DefaultJumpPower or 50
        local dh = config.DefaultJumpHeight or 7.2
        pcall(function()
            hum.UseJumpPower = true
            hum.JumpPower = dj
            if hum.JumpHeight ~= nil then hum.JumpHeight = dh end
        end)
    end
end


SlideBoostConn = SlideBoostConn or nil
function StartSlideBoostLoop()
    if SlideBoostConn then
        pcall(function() SlideBoostConn:Disconnect() end)
        SlideBoostConn = nil
    end
    SlideBoostConn = RunService.Heartbeat:Connect(function(dt)
        if not config.SlideBoostEnabled then return end
        if not isSlideKeyDown() then return end
        local hum, hrp = getHum(), getHRP()
        if not hum or not hrp then return end
        if hum.Health <= 0 then return end

        local mult = tonumber(config.SlideBoostMult) or 2.5
        local base = tonumber(config.SlideBaseSpeed) or 0
        -- velocidad objetivo
        local dir = getMoveDir()
        if dir.Magnitude < 0.05 then
            -- si no hay input, usar look horizontal
            local lv = hrp.CFrame.LookVector
            dir = Vector3.new(lv.X, 0, lv.Z)
            if dir.Magnitude < 0.05 then return end
            dir = dir.Unit
        end

        local walk = (config.SpeedEnabled and (config.WalkSpeed or 50)) or (hum.WalkSpeed > 1 and hum.WalkSpeed or 16)
        local targetSpeed = math.max(walk * mult, base > 0 and base or 0)
        -- clamp razonable anti-kick
        targetSpeed = math.clamp(targetSpeed, 16, 220)

        local vel = hrp.AssemblyLinearVelocity
        local target = dir * targetSpeed
        -- boost horizontal, conservar Y (salto/gravedad)
        local blend = math.clamp((dt or 0.016) * 14, 0.15, 1)
        local nx = vel.X + (target.X - vel.X) * blend
        local nz = vel.Z + (target.Z - vel.Z) * blend
        hrp.AssemblyLinearVelocity = Vector3.new(nx, vel.Y, nz)

        -- también subir WalkSpeed temporalmente mientras slides
        pcall(function()
            if hum.WalkSpeed < targetSpeed * 0.85 then
                hum.WalkSpeed = targetSpeed * 0.9
            end
        end)
    end)
end

function StopSlideBoostLoop()
    if SlideBoostConn then
        pcall(function() SlideBoostConn:Disconnect() end)
        SlideBoostConn = nil
    end
end

function UpdateMovement()

    local hum, hrp = getHum(), getHRP()
    if not hum or not hrp then return end

    -- JUMP: solo si toggle ON; si OFF restaura
    if config.JumpPowerEnabled then
        ApplyJumpState()
    end

    -- SPEED: solo velocity/walk si toggle ON
    if config.SpeedEnabled then
        local spd = config.WalkSpeed or 50
        pcall(function() hum.WalkSpeed = spd end)
        if config.SpeedVelocityMode ~= false then
            local dir = getMoveDir()
            if dir.Magnitude > 0.05 then
                local vel = hrp.AssemblyLinearVelocity
                local target = dir * spd
                hrp.AssemblyLinearVelocity = Vector3.new(target.X, vel.Y, target.Z)
            end
        end
    end

    -- Slide boost tiene su propio Heartbeat (StartSlideBoostLoop)

    if config.AntiFling and not (config.SlideBoostEnabled and isSlideKeyDown()) then
        hrp.AssemblyAngularVelocity = Vector3.zero
        local maxV = (config.SpeedEnabled and (config.WalkSpeed or 50) * 4) or 220
        if hrp.AssemblyLinearVelocity.Magnitude > math.max(maxV, 220) then
            hrp.AssemblyLinearVelocity = hrp.AssemblyLinearVelocity.Unit * math.max(maxV, 220)
        end
    end
    if config.DesyncJitter and not config.Enabled and not IsAntiUnnamedActive() and not config.VoidSpamEnabled and not config.RagebotEnabled then
        local p = config.DesyncJitterPower or 12
        hrp.CFrame = hrp.CFrame * CFrame.new((math.random() - 0.5) * p * 0.05, (math.random() - 0.5) * p * 0.02, (math.random() - 0.5) * p * 0.05)
    end
    AutoEquipMelee()
    ApplyCharVisuals()
end

local StartRagebot, StopRagebot, StartAutoClick, StopAutoClick
local StartNoRecoil, StopNoRecoil, StartAntiUnnamed, StopAntiUnnamed
local ArmAntiUnnamedOnSpawn, StartAUWatcher
local StartFly, StopFly, StartNoclip, StopNoclip, StartAntiAim, StopAntiAim
local StartMainLoop, StartStickRender, StartESP, StopESP, StartAimbot, StopAimbot

StartNoRecoil = function()
    SafeDisconnect("NoRecoil")
    -- solo ItemLibrary, CERO camara
    if config.RapidFire then
        ApplyItemLibraryGuns(true)
    else
        local Items = getItems()
        if Items then
            for name, data in pairs(Items) do
                if typeof(data) == "table" and not gunExceptions[name] then
                    ilSet(name, data, "ShootSpread", 0)
                    ilSet(name, data, "ShootRecoil", 0)
                    ilSet(name, data, "Recoil", 0)
                    ilSet(name, data, "Spread", 0)
                end
            end
        end
    end
end
StopNoRecoil = function()
    SafeDisconnect("NoRecoil")
    if not config.RapidFire then
        local Items = getItems()
        if Items then
            for name, data in pairs(Items) do
                if typeof(data) == "table" then ilRestore(name, data) end
            end
            if config.FastMelee then ApplyItemLibraryMelee(true) end
        end
    end
end

function GetAimbotTarget()
    local closest, shortest = nil, config.AimbotFOV
    local mousePos = UserInputService:GetMouseLocation()
    local partName = config.AimbotPart or "Head"
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if config.AimbotTeamCheck and IsSameTeam(plr) then continue end
        local char = plr.Character
        if not char then continue end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local part = char:FindFirstChild(partName) or char:FindFirstChild("Head")
        if not hum or not part or hum.Health <= 0 then continue end
        local sp, on = Camera:WorldToViewportPoint(part.Position)
        if not on or sp.Z < 0 then continue end
        local d = (Vector2.new(sp.X, sp.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
        if d < shortest then shortest, closest = d, part end
    end
    return closest
end
-- Aimbot (logica simple: hold RMB + CFrame.lookAt, con FOV/team/wall opcionales)
local aimbotHolding = false

function isEnemyForAimbot(plr)
    if plr == LocalPlayer then return false end
    if not config.AimbotTeamCheck then return true end
    return IsEnemy(plr)
end

function aimbotWallClear(fromPos, toPos, targetChar)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local filter = {}
    if LocalPlayer.Character then table.insert(filter, LocalPlayer.Character) end
    if targetChar then table.insert(filter, targetChar) end
    local vm = workspace:FindFirstChild("ViewModels")
    if vm then table.insert(filter, vm) end
    params.FilterDescendantsInstances = filter
    params.IgnoreWater = true
    local delta = toPos - fromPos
    local dist = delta.Magnitude
    if dist < 0.5 then return true end
    local res = workspace:Raycast(fromPos, delta.Unit * dist, params)
    if not res then return true end
    if targetChar and res.Instance and res.Instance:IsDescendantOf(targetChar) then return true end
    return false
end

function GetAimbotTargetRivals()
    local cam = workspace.CurrentCamera
    if not cam then return nil end
    local partName = config.AimbotPart or "Head"
    local fov = tonumber(config.AimbotFOV) or 400 -- PIXELES desde el centro
    local center = UserInputService:GetMouseLocation()
    -- si no hay mouse (gamepad), centro pantalla
    if not center or center.X == 0 then
        center = Vector2.new(cam.ViewportSize.X * 0.5, cam.ViewportSize.Y * 0.5)
    end
    local bestPart, bestScore = nil, math.huge
    local camPos = cam.CFrame.Position

    for _, plr in pairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if config.AimbotTeamCheck and not IsEnemy(plr) then continue end
        local char = plr.Character
        if not char then continue end
        local head = char:FindFirstChild(partName) or char:FindFirstChild("Head")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not head or not hum or hum.Health <= 0 then continue end

        local sp, onScreen = cam:WorldToViewportPoint(head.Position)
        if sp.Z <= 0 then continue end

        local screenDist = (Vector2.new(sp.X, sp.Y) - Vector2.new(center.X, center.Y)).Magnitude
        if screenDist > fov then continue end

        if config.AimbotWallCheck or config.AimbotVisibleCheck then
            if not aimbotWallClear(camPos, head.Position, char) then continue end
        end

        local score = screenDist
        if score < bestScore then
            bestScore = score
            bestPart = head
        end
    end
    return bestPart
end

StartAimbot = function()
    SafeDisconnect("Aimbot")
    SafeDisconnect("AimbotInput")
    SafeDisconnect("AimbotInputEnd")
    pcall(function() RunService:UnbindFromRenderStep("__as_aimbot") end)
    pcall(function() RunService:UnbindFromRenderStep("__as_aimbot2") end)
    aimbotHolding = false

    Connections.AimbotInput = UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            aimbotHolding = true
        end
    end)
    Connections.AimbotInputEnd = UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            aimbotHolding = false
        end
    end)

    local function isAimingHold()
        if aimbotHolding then return true end
        local ok, down = pcall(function()
            return UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
        end)
        return ok and down == true
    end

    local function aimStep()
        if not config.AimbotEnabled then return end
        if IsAntiUnnamedActive() then return end

        if config.AimbotKeyHold ~= false and not isAimingHold() then
            return
        end

        local cam = workspace.CurrentCamera
        if not cam then return end

        local target = GetAimbotTargetRivals()
        if not target or not target.Parent then return end

        local aimPos = target.Position
        local screenPos, onScreen = cam:WorldToViewportPoint(aimPos)
        if not onScreen and screenPos.Z < 0 then return end

        -- 1) Mover mouse hacia el target (funciona mejor en Rivals que pelear CFrame)
        local movedMouse = false
        pcall(function()
            local mouse = UserInputService:GetMouseLocation()
            local dx = screenPos.X - mouse.X
            local dy = screenPos.Y - mouse.Y
            local smooth = tonumber(config.AimbotSmooth) or 0.9
            local sens = math.clamp(smooth, 0.2, 1)
            dx, dy = dx * sens, dy * sens
            if math.abs(dx) < 0.5 and math.abs(dy) < 0.5 then return end
            if mousemoverel then
                mousemoverel(dx, dy)
                movedMouse = true
            elseif Input and Input.MouseMove then
                Input.MouseMove(dx, dy)
                movedMouse = true
            end
        end)

        -- 2) Fallback camara si el executor no tiene mousemoverel
        if not movedMouse then
            local pos = cam.CFrame.Position
            cam.CFrame = CFrame.new(pos, aimPos)
        end
    end

    -- Heartbeat + RenderStep (una vez cada uno)
    local aimFrame = 0
    Connections.Aimbot = RunService.RenderStepped:Connect(function(...)
        aimFrame += 1
        if aimFrame % 2 ~= 0 then return end -- OPT ~30fps aim
        return aimStep(...)
    end)
    Notify("Aimbot", "ON · hold click derecho")
end

StopAimbot = function()
    SafeDisconnect("Aimbot")
    SafeDisconnect("AimbotInput")
    SafeDisconnect("AimbotInputEnd")
    pcall(function() RunService:UnbindFromRenderStep("__as_aimbot") end)
    pcall(function() RunService:UnbindFromRenderStep("__as_aimbot2") end)
    aimbotHolding = false
end

StartRagebot = function()
    config.RageUseItem = true -- force always with ragebot
    SafeDisconnect("Ragebot")
    RagebotTarget = nil
    voidRagePhase = 0 -- empezar en VACIO
    lastVoidRage = tick()
    if not modulesOK then
        loadRivalsModules()
    end
    Connections.Ragebot = RunService.Heartbeat:Connect(function()
        if not config.RagebotEnabled or not CanDoDangerousMove() or IsAntiUnnamedActive() then return end
        if config.VoidSpamEnabled and not config.VoidRage then return end
        if not modulesOK then
            if tick() - (getgenv()._ShakoRageModTry or 0) > 2.5 then
                getgenv()._ShakoRageModTry = tick()
                pcall(function()
                    if not util then util = require(RS.Modules.Utility) end
                    if not enum then enum = require(RS.Modules.EnumLibrary) end
                    if not FighterController then
                        local ps = LocalPlayer:FindFirstChild("PlayerScripts")
                        local c = ps and ps:FindFirstChild("Controllers")
                        local fc = c and c:FindFirstChild("FighterController")
                        if fc then FighterController = require(fc) end
                    end
                    modulesOK = (util ~= nil) and (enum ~= nil)
                end)
            end
            if not modulesOK then return end
        end

        updateDeflection()
        EnsureCameraFree()

        local target = GetRagebotTarget()
        -- prefer rivals team check
        if target and not isEnemyRageRivals(target) then
            RagebotTarget = nil
            return
        end
        if not target or not target.Character then
            RagebotTarget = nil
            voidRagePhase = 0
            local my = getHRP()
            if my and (config.RageVoidFirst ~= false or config.VoidRage) then
                local dist = tonumber(config.VoidRageAwayDist) or 99999999
                pcall(function() VoidServerMove(my.Position + Vector3.new(0, dist, 0)) end)
            end
            return
        end
        local head = target.Character:FindFirstChild("Head")
        local root = target.Character:FindFirstChild("HumanoidRootPart")
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        -- target muerto → void y siguiente
        if not head or not root or not hum or hum.Health <= 0 then
            RagebotTarget = nil
            if config.RageVoidOnKill ~= false or config.RageVoidFirst or config.VoidRage then
                voidRagePhase = 0 -- VOID
                lastVoidRage = tick()
                local my = getHRP()
                if my then
                    local off = VoidRageOffset(root or my)
                    -- dist enorme al vacio
                    local dist = tonumber(config.VoidRageAwayDist) or 99999999
                    if off.Magnitude < dist * 0.5 then
                        off = Vector3.new(0, dist, 0)
                    end
                    pcall(function() VoidServerMove(my.Position + off) end)
                end
            end
            return
        end
        RagebotTarget = target

        if config.RageSkipDeflect and deflecting[target] then return end

        local lookPos = PredictPos(root, head)

        -- SIEMPRE arriba de la persona mirando a la cabeza
        local aboveY = tonumber(config.RagebotHeight) or 2.2
        if hasKnifeViewModel(target) then
            aboveY = math.max(aboveY, tonumber(config.RageDesyncKnifeY) or 6)
        else
            aboveY = math.max(aboveY, tonumber(config.RageDesyncNormalY) or 2.2)
        end
        local ox = tonumber(config.RagebotOffsetX) or 0
        local oz = tonumber(config.RagebotOffsetZ) or 0
        local desyncPos = head.Position + Vector3.new(ox, aboveY, oz)
        local desyncCF = CFrame.lookAt(desyncPos, head.Position)

        -- CICLO: VOID (0) → ATTACK (1) → al matar VOID otra vez
        local useVoidCycle = config.RageVoidFirst ~= false or config.VoidRage == true
        if useVoidCycle then
            -- min hide: no TP hiper-rapido (queda mal)
            local hideT = math.max(tonumber(config.RageVoidHideTime) or 0.35, 0.28)
            local atkT = math.max(tonumber(config.RageVoidAttackTime) or 0.45, 0.15)
            if voidRagePhase == 0 then
                -- HIDE: vacio — NO DISPARA / no gasta balas
                local dist = tonumber(config.VoidRageAwayDist) or 99999999
                local off = VoidRageOffset(root)
                if off.Magnitude < 1000 then
                    off = Vector3.new(0, dist, 0)
                end
                VoidServerMove(CFrame.lookAt(root.Position + off, lookPos))
                if tick() - (lastVoidRage or 0) >= hideT then
                    voidRagePhase = 1
                    lastVoidRage = tick()
                end
                return -- crítico: sin fireRageUseItem en void
            elseif tick() - (lastVoidRage or 0) >= atkT then
                voidRagePhase = 0
                lastVoidRage = tick()
                return -- tampoco dispara al salir
            else
                -- ATTACK phase
                if config.RageDesync then
                    local myRoot = getHRP()
                    if myRoot then
                        local oldCF = myRoot.CFrame
                        local oldVel = myRoot.AssemblyLinearVelocity
                        local oldRot = myRoot.AssemblyAngularVelocity
                        pcall(function() RunService:UnbindFromRenderStep("__as_rage_restore") end)
                        myRoot.CFrame = desyncCF
                        RunService:BindToRenderStep("__as_rage_restore", Enum.RenderPriority.Camera.Value - 1, function()
                            if myRoot and myRoot.Parent then
                                myRoot.CFrame = oldCF
                                myRoot.AssemblyLinearVelocity = oldVel
                                myRoot.AssemblyAngularVelocity = oldRot
                            end
                            pcall(function() RunService:UnbindFromRenderStep("__as_rage_restore") end)
                        end)
                    end
                else
                    SafeEnemyCFrame(root, config.RagebotHeight or 2.2, config.RagebotOffsetX or 0, config.RagebotOffsetZ or 0, lookPos)
                end
            end
        elseif config.RageDesync then
            local myRoot = getHRP()
            if myRoot then
                local oldCF = myRoot.CFrame
                local oldVel = myRoot.AssemblyLinearVelocity
                local oldRot = myRoot.AssemblyAngularVelocity
                pcall(function() RunService:UnbindFromRenderStep("__as_rage_restore") end)
                myRoot.CFrame = desyncCF
                RunService:BindToRenderStep("__as_rage_restore", Enum.RenderPriority.Camera.Value - 1, function()
                    if myRoot and myRoot.Parent then
                        myRoot.CFrame = oldCF
                        myRoot.AssemblyLinearVelocity = oldVel
                        myRoot.AssemblyAngularVelocity = oldRot
                    end
                    pcall(function() RunService:UnbindFromRenderStep("__as_rage_restore") end)
                end)
            end
        else
            SafeEnemyCFrame(root, config.RagebotHeight or 2.2, config.RagebotOffsetX or 0, config.RagebotOffsetZ or 0, lookPos)
        end

        if config.RageLookSyncOnShoot then
            SyncAimOnShoot(lookPos)
        elseif config.RagebotLookAtHead or config.CameraLookAtTarget then
            LookAtPos(lookPos)
        end

        -- UseItem fire: SOLO en fase attack + con target vivo
        if true then
            if useVoidCycle and voidRagePhase ~= 1 then return end
            if not target or not head or not root then return end
            if not hum or hum.Health <= 0 then return end
            local lf = FighterController and FighterController.LocalFighter
            local item = lf and lf.EquippedItem
            if not item then return end
            -- Attack speed estilo UE: 1 = lento, 20 = casi insta
            local atk = tonumber(config.RageAttackSpeed) or 10
            atk = math.clamp(atk, 1, 20)
            local rate = tonumber(config.RageFireRate) or 0
            if rate <= 0 then
                -- interval minimo entre tps/shots segun attack
                rate = 0.08 / atk  -- 20 → 0.004s, 10 → 0.008s, 1 → 0.08s
            else
                rate = rate / atk
            end
            if (tick() - rageLastFire) < rate then return end
            rageLastFire = tick()

            local hitPart = head
            if config.RageMultipoint then
                local torso = target.Character:FindFirstChild("UpperTorso")
                    or target.Character:FindFirstChild("Torso")
                -- resolver: si head muy lejos del root (desync enemigo), apunta torso
                if config.RageResolver and torso then
                    local hd = (head.Position - root.Position).Magnitude
                    if hd > 6 then hitPart = torso end
                end
            end

            local shots = math.clamp(tonumber(config.RageShotsPerTick) or 3, 1, 6)
            for _ = 1, shots do
                fireRageUseItem(item, desyncCF, hitPart, root)
            end
            -- multipoint: 1 shot extra a torso
            if config.RageMultipoint then
                local torso = target.Character:FindFirstChild("UpperTorso")
                    or target.Character:FindFirstChild("Torso")
                if torso and torso ~= hitPart then
                    fireRageUseItem(item, desyncCF, torso, root)
                end
            end
        end
    end)
end
StopRagebot = function()
    SafeDisconnect("Ragebot")
    RagebotTarget = nil
    pcall(function() RunService:UnbindFromRenderStep("__as_rage_restore") end)
    pcall(function() RunService:UnbindFromRenderStep("__as_void_desync") end)
    pcall(function() if RageInd then RageInd.hide() end end)
end

-- ==================== RAGE INDICATOR (Grief.cc style) ====================
-- Círculo en el target + snapline + texto "ragebot: attacking / idle"
local RageInd = {
    killStart = nil,
    lastTarget = nil,
}
do
    local ind, indOut, snap, snapOut, line1, line2
    pcall(function()
        ind = Drawing.new("Circle")
        ind.Thickness = 1.5
        ind.NumSides = 36
        ind.Filled = false
        ind.Transparency = 1
        ind.Visible = false
        ind.Radius = 12
        ind.Color = Color3.fromRGB(255, 50, 50)

        indOut = Drawing.new("Circle")
        indOut.Thickness = 4
        indOut.NumSides = 36
        indOut.Filled = false
        indOut.Transparency = 1
        indOut.Visible = false
        indOut.Radius = 12
        indOut.Color = Color3.fromRGB(0, 0, 0)

        snap = Drawing.new("Line")
        snap.Visible = false
        snap.Thickness = 1.5
        snap.Transparency = 1
        snap.Color = Color3.fromRGB(0, 186, 255)

        snapOut = Drawing.new("Line")
        snapOut.Visible = false
        snapOut.Thickness = 3.5
        snapOut.Transparency = 1
        snapOut.Color = Color3.fromRGB(0, 0, 0)

        local function mkTxt()
            local t = Drawing.new("Text")
            t.Size = 14
            t.Font = 2
            t.Outline = true
            t.Center = true
            t.Visible = false
            t.Color = Color3.fromRGB(235, 235, 235)
            pcall(function() t.OutlineColor = Color3.fromRGB(0, 0, 0) end)
            return t
        end
        line1 = mkTxt()
        line2 = mkTxt()
    end)

    local COL = {
        Red = Color3.fromRGB(255, 50, 50),
        Purple = Color3.fromRGB(180, 0, 255),
        Cyan = Color3.fromRGB(0, 186, 255),
        White = Color3.fromRGB(235, 235, 235),
        Green = Color3.fromRGB(80, 255, 140),
    }

    function RageInd.hide()
        pcall(function()
            if ind then ind.Visible = false end
            if indOut then indOut.Visible = false end
            if snap then snap.Visible = false end
            if snapOut then snapOut.Visible = false end
            if line1 then line1.Visible = false end
            if line2 then line2.Visible = false end
        end)
        RageInd.killStart = nil
        RageInd.lastTarget = nil
    end

    local function statusText(target)
        if not config.RagebotEnabled then
            return "ragebot: off", ""
        end
        if not target or not target.Character then
            RageInd.killStart = nil
            return "ragebot: idle", ""
        end
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        local hp = hum and hum.Health or 0
        local name = target.DisplayName or target.Name or "target"
        if hp <= 0 then
            RageInd.killStart = nil
            return "ragebot: idle", ""
        end
        -- attacking
        if not RageInd.killStart or RageInd.lastTarget ~= target then
            RageInd.killStart = tick()
            RageInd.lastTarget = target
        end
        local elapsed = tick() - (RageInd.killStart or tick())
        local main = string.format("ragebot : attacking (%s) · %.1fs", name, elapsed)
        local detail = string.format("%.0f hp", hp)
        if config.VoidRage then
            detail = detail .. " · void"
        end
        if config.RageDesync then
            detail = detail .. " · desync"
        end
        return main, detail
    end

    function RageInd.update()
        if not ind then return end
        local showInd = config.RageIndicatorEnabled ~= false
        local showStatus = config.RageStatusEnabled ~= false
        local showSnap = config.RageIndicatorSnapline ~= false
        if not config.RagebotEnabled then
            RageInd.hide()
            return
        end
        if not showInd and not showStatus and not showSnap then
            RageInd.hide()
            return
        end

        local cam = workspace.CurrentCamera
        if not cam then RageInd.hide() return end

        local target = RagebotTarget
        local part = nil
        if target and target.Character then
            part = target.Character:FindFirstChild("Head")
                or target.Character:FindFirstChild("HumanoidRootPart")
        end

        -- status text (siempre visible si status on)
        if showStatus and line1 then
            local main, detail = statusText(target)
            local anchor
            if config.RageStatusMode == "crosshair" then
                local m = UserInputService:GetMouseLocation()
                anchor = Vector2.new(m.X, m.Y + (config.RageStatusOffsetY or 40))
            else
                local vs = cam.ViewportSize
                anchor = Vector2.new(vs.X * 0.5, vs.Y * 0.5 + (config.RageStatusOffsetY or 48))
            end
            line1.Text = main
            line1.Position = anchor
            line1.Visible = true
            if detail ~= "" and line2 then
                line2.Text = detail
                line2.Position = anchor + Vector2.new(0, 16)
                line2.Visible = true
            elseif line2 then
                line2.Visible = false
            end
        else
            if line1 then line1.Visible = false end
            if line2 then line2.Visible = false end
        end

        if not part then
            if ind then ind.Visible = false end
            if indOut then indOut.Visible = false end
            if snap then snap.Visible = false end
            if snapOut then snapOut.Visible = false end
            return
        end

        local sp, onScreen = cam:WorldToViewportPoint(part.Position)
        if not onScreen or sp.Z <= 0 then
            if ind then ind.Visible = false end
            if indOut then indOut.Visible = false end
            if snap then snap.Visible = false end
            if snapOut then snapOut.Visible = false end
            return
        end
        local pos = Vector2.new(sp.X, sp.Y)
        local colName = config.RageIndicatorColor or "Red"
        local col = COL[colName] or COL.Red
        -- purple si desync activo
        if config.RageDesync then
            col = COL.Purple
        end

        if showInd then
            -- pulse radius
            local pulse = 11 + 2 * math.sin(tick() * 6)
            indOut.Position = pos
            indOut.Radius = pulse
            indOut.Visible = true
            ind.Position = pos
            ind.Radius = pulse
            ind.Color = col
            ind.Visible = true
        else
            ind.Visible = false
            indOut.Visible = false
        end

        if showSnap then
            local from
            if config.RageStatusMode == "crosshair" then
                local m = UserInputService:GetMouseLocation()
                from = Vector2.new(m.X, m.Y)
            else
                local vs = cam.ViewportSize
                from = Vector2.new(vs.X * 0.5, vs.Y * 0.5)
            end
            snapOut.From = from
            snapOut.To = pos
            snapOut.Visible = true
            snap.From = from
            snap.To = pos
            snap.Color = col
            snap.Visible = true
        else
            snap.Visible = false
            snapOut.Visible = false
        end
    end
end
getgenv().RageInd = RageInd

-- loop del indicator (barato)
if not Connections then Connections = {} end
pcall(function()
    if Connections.RageInd then Connections.RageInd:Disconnect() end
    local riN = 0
    Connections.RageInd = RunService.Heartbeat:Connect(function()
        if not config.RagebotEnabled and not config.RageIndicatorEnabled then return end
        riN = riN + 1
        if riN % 2 == 1 then return end -- ~30fps
        pcall(function() if RageInd and RageInd.update then RageInd.update() end end)
    end)
end)


StartAutoClick = function()
    SafeDisconnect("AutoClick")
    LastClick = 0
    Connections.AutoClick = RunService.RenderStepped:Connect(function()
        if not config.AutoClickEnabled or UserInputService:GetFocusedTextBox() then return end
        if config.AutoClickOnlyOnTarget then
            if not (config.RagebotEnabled and RagebotTarget) and not (config.Enabled and CurrentTarget) then return end
        end
        if tick() - LastClick >= 1 / math.max(config.AutoClickCPS, 1) then
            LastClick = tick()
            pcall(function() mouse1click() end)
        end
    end)
end
StopAutoClick = function() SafeDisconnect("AutoClick") end

StartNoclip = function()
    SafeDisconnect("Noclip")
    local _ncAcc = 0
    Connections.Noclip = RunService.Heartbeat:Connect(function(dt)
        if not config.NoclipEnabled then return end
        _ncAcc = _ncAcc + dt
        if _ncAcc < 0.08 then return end -- ~12/s max
        _ncAcc = 0
        local char = LocalPlayer.Character
        if not char then return end
        for _, p in ipairs(char:GetChildren()) do
            if p:IsA("BasePart") then
                p.CanCollide = false
            elseif p:IsA("Accessory") then
                local h = p:FindFirstChild("Handle")
                if h and h:IsA("BasePart") then h.CanCollide = false end
            end
        end
    end)
end
StopNoclip = function() SafeDisconnect("Noclip") end

StartAntiAim = function()
    SafeDisconnect("AntiAim")
    pcall(function() RunService:UnbindFromRenderStep("__as_aa_client") end)
    local ug_next, ug_yaw = 0, 0

    -- VOS: cuerpo/camara normales (no ves underground)
    RunService:BindToRenderStep("__as_aa_client", Enum.RenderPriority.Camera.Value - 1, function()
        if not config.AntiAimEnabled then return end
        local myHRP = getHRP()
        if not myHRP then return end
        local cam = workspace.CurrentCamera
        if not cam then return end
        local pos = myHRP.Position
        local lv = cam.CFrame.LookVector
        local yaw = math.atan2(-lv.X, -lv.Z)
        pcall(function()
            myHRP.CFrame = CFrame.new(pos) * CFrame.Angles(0, yaw, 0)
        end)
    end)

    local function buildServerCF(myHRP)
        local mode = config.AntiAimMode
        local isUG = (mode == "Underground" or config.AntiAimUnderground)
        -- con rage solo Underground (y modos de orientacion si se quiere)
        if config.RagebotEnabled and not isUG then
            return nil
        end
        local pitch = math.rad(config.AAPitch or 0)
        local inv = config.AAInvert and -1 or 1
        -- POSICION actual (la del rage si esta ON) — solo cambiamos angulos
        local pos = myHRP.Position
        local serverCF = nil

        if isUG then
            local now = tick()
            local basePitch = tonumber(config.UndergroundPitch)
            if not basePitch then
                local legacy = tonumber(config.AntiAimUndergroundY) or -25
                basePitch = (legacy < -90) and -25 or math.clamp(legacy, -89, 0)
            end
            basePitch = math.clamp(basePitch, -89, -5)
            local pitchJ = basePitch
            if config.UndergroundRandom ~= false then
                pitchJ = basePitch + (math.noise(now * 2.1) * 8)
            end
            local jAmt = tonumber(config.UndergroundJitter) or 35
            if now >= ug_next then
                ug_yaw = (math.random() * 2 - 1) * jAmt
                ug_next = now + ((config.UndergroundRandom ~= false) and (0.05 + math.random() * 0.12) or 0.08)
            end
            SpinAngle = (SpinAngle or 0) + (config.SpinSpeed or 20) * (tonumber(config.UndergroundSpin) or 0.4) * 0.15 * inv
            local yaw = math.rad((SpinAngle % 360) + ug_yaw * inv)
            serverCF = CFrame.new(pos) * CFrame.Angles(math.rad(pitchJ), yaw, 0)
        elseif mode == "Spin" then
            SpinAngle = (SpinAngle + config.SpinSpeed * 0.15 * inv) % 360
            serverCF = CFrame.new(pos) * CFrame.Angles(pitch, math.rad(SpinAngle), 0)
        elseif mode == "Jitter" then
            JitterSide = -JitterSide
            serverCF = CFrame.new(pos) * CFrame.Angles(pitch, math.rad(config.JitterAngle * JitterSide * inv), 0)
        elseif mode == "Random" then
            serverCF = CFrame.new(pos) * CFrame.Angles(pitch, math.rad(math.random(0, 360)), 0)
        elseif mode == "Backwards" then
            local look = Camera.CFrame.LookVector
            serverCF = CFrame.new(pos, pos - Vector3.new(look.X, 0, look.Z)) * CFrame.Angles(pitch, 0, 0)
        elseif mode == "Sideways" then
            local look = Camera.CFrame.LookVector
            local right = Vector3.new(-look.Z, 0, look.X)
            serverCF = CFrame.new(pos, pos + right * inv) * CFrame.Angles(pitch, 0, 0)
        elseif mode == "ManualYaw" then
            serverCF = CFrame.new(pos) * CFrame.Angles(pitch, math.rad(config.ManualYaw or 180), 0)
        end
        return serverCF
    end

    -- SERVER / OTROS: PostSimulation = DESPUES del rage (rage mueve, AA solo rota ahi)
    local postOk = false
    pcall(function()
        Connections.AntiAim = RunService.PostSimulation:Connect(function()
            if not config.AntiAimEnabled or not CanDoDangerousMove() then return end
            if config.Enabled then return end
            if IsAntiUnnamedActive and IsAntiUnnamedActive() then return end
            if config.VoidSpamEnabled then return end
            local myHRP = getHRP()
            if not myHRP then return end
            local serverCF = buildServerCF(myHRP)
            if serverCF then
                pcall(function() myHRP.CFrame = serverCF end)
            end
        end)
        postOk = true
    end)
    if not postOk then
        -- fallback Heartbeat (ejecutar tarde)
        Connections.AntiAim = RunService.Heartbeat:Connect(function()
            if not config.AntiAimEnabled or not CanDoDangerousMove() then return end
            if config.Enabled then return end
            if IsAntiUnnamedActive and IsAntiUnnamedActive() then return end
            if config.VoidSpamEnabled then return end
            local myHRP = getHRP()
            if not myHRP then return end
            local serverCF = buildServerCF(myHRP)
            if serverCF then
                pcall(function() myHRP.CFrame = serverCF end)
            end
        end)
    end
end
StopAntiAim = function()
    SafeDisconnect("AntiAim")
    pcall(function() RunService:UnbindFromRenderStep("__as_aa_client") end)
end

-- ==================== UE COUNTER (solo esto, sin Anti Unnamed viejo) ====================
StartAntiUnnamed = function()
    StartUECounter()
end
StopAntiUnnamed = function()
    StopUECounter()
end
ArmAntiUnnamedOnSpawn = function()
    if config.UECounterEnabled or config.AntiUnnamed then
        task.defer(function()
            task.wait(0.3)
            if config.UECounterEnabled or config.AntiUnnamed then
                StartUECounter()
            end
        end)
    end
end
StartAUWatcher = function()
    SafeDisconnect("AUWatch")
    if not (config.UECounterEnabled or config.AntiUnnamed) then return end
    Connections.AUWatch = LocalPlayer.CharacterAdded:Connect(function()
        if config.UECounterEnabled or config.AntiUnnamed then
            ArmAntiUnnamedOnSpawn()
        end
    end)
end

function StartUECounter()
    SafeDisconnect("AntiUnnamed")
    config.UECounterEnabled = true
    config.AntiUnnamed = true
    UECounterReady = true

    local function ueTp()
        if not (config.UECounterEnabled or config.AntiUnnamed) then return end
        local char = LocalPlayer.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local y = tonumber(config.UECounterY) or 500
        pcall(function()
            VoidServerMove(hrp.Position + Vector3.new(0, y, 0))
        end)
    end

    Connections.AntiUnnamed = task.spawn(function()
        while config.UECounterEnabled or config.AntiUnnamed do
            ueTp()
            local cd = tonumber(config.UECounterCooldown) or 0.2
            if cd < 0 then cd = 0 end
            if cd > 2 then cd = 2 end
            task.wait(cd)
        end
        Connections.AntiUnnamed = nil
    end)
    if not getgenv()._ShakoQuiet then
        Notify("UE Counter", "ON · Y +" .. tostring(config.UECounterY or 500))
    end
end

function StopUECounter()
    config.UECounterEnabled = false
    config.AntiUnnamed = false
    UECounterReady = false
    SafeDisconnect("AntiUnnamed")
    SafeDisconnect("AUWatch")
    pcall(function() RunService:UnbindFromRenderStep("__as_au_lock") end)
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then pcall(function() hrp.Anchored = false end) end
    if not getgenv()._ShakoQuiet and not (getgenv()._ShakoBootAt and tick() - getgenv()._ShakoBootAt < 8) then
        Notify("UE Counter", "OFF")
    end
end

-- anti Sleepy multi-pos: cambia direccion (arriba / lados / atras / frente)
local STICK_POS = {"Under", "Above", "Right", "Left", "Behind", "Front", "RightHand", "LeftHand"}

function getStickOffsetCF(root, head)
    local away = math.max(tonumber(config.StickAway) or 6, 0.8)
    if config.StickFarMode then away = away * (tonumber(config.StickFarMult) or 2.5) end
    local baseY = tonumber(config.StickHeight) or -2.8
    local ox = tonumber(config.StickOffsetX) or 0
    local oz = tonumber(config.StickOffsetZ) or 0

    if config.StickMultiPos ~= false then
        if tick() - stickPosLast >= (config.StickPosInterval or 0.18) then
            stickPosLast = tick()
            stickPosIndex = stickPosIndex % #STICK_POS + 1
        end
    end
    local mode = STICK_POS[stickPosIndex] or "Under"
    local cf = root.CFrame
    local look = (head and head.Position) or root.Position
    local pos
    if mode == "Under" then
        pos = (cf * CFrame.new(ox, baseY, oz)).Position
    elseif mode == "Above" then
        pos = (cf * CFrame.new(ox, math.max(baseY, 0) + 2.2 + away * 0.35, oz - 0.4)).Position
    elseif mode == "Right" then
        pos = (cf * CFrame.new(away + ox, baseY + 0.5, oz)).Position
    elseif mode == "Left" then
        pos = (cf * CFrame.new(-(away + ox), baseY + 0.5, oz)).Position
    elseif mode == "Behind" then
        pos = (cf * CFrame.new(ox, baseY + 0.3, away + oz)).Position
    elseif mode == "Front" then
        pos = (cf * CFrame.new(ox, baseY + 0.3, -(away * 0.7) + oz)).Position
    elseif mode == "RightHand" then
        pos = (cf * CFrame.new(away * 0.85 + ox, baseY + 1.0, -0.5 + oz)).Position
    else -- LeftHand
        pos = (cf * CFrame.new(-(away * 0.85 + ox), baseY + 1.0, -0.5 + oz)).Position
    end
    return CFrame.lookAt(pos, look), pos, look, mode
end

function StickToTarget(target)
    if IsAntiUnnamedActive() or config.VoidSpamEnabled or config.RagebotEnabled then return false end
    if not target or not target.Character then return false end
    -- NUNCA pegarse al team
    if not IsEnemy(target) then return false end
    local hum = target.Character:FindFirstChildOfClass("Humanoid")
    local root = target.Character:FindFirstChild("HumanoidRootPart")
    local head = target.Character:FindFirstChild("Head")
    if not hum or not root or hum.Health <= 0 then return false end

    local my = getHRP()
    if not my then return false end

    local stickCF, stickPos, lookPos, mode = getStickOffsetCF(root, head)

    -- DESYNC igual que rage: server en stick pos, local no ve el TP
    if config.StickDesync ~= false then
        local oldCF = my.CFrame
        local oldVel = my.AssemblyLinearVelocity
        local oldRot = my.AssemblyAngularVelocity
        pcall(function() RunService:UnbindFromRenderStep("__as_stick_restore") end)
        my.CFrame = stickCF
        if config.ForceZeroVelocity then
            my.AssemblyLinearVelocity = Vector3.zero
            my.AssemblyAngularVelocity = Vector3.zero
        end
        RunService:BindToRenderStep("__as_stick_restore", Enum.RenderPriority.Camera.Value - 1, function()
            if my and my.Parent then
                my.CFrame = oldCF
                my.AssemblyLinearVelocity = oldVel
                my.AssemblyAngularVelocity = oldRot
            end
            pcall(function() RunService:UnbindFromRenderStep("__as_stick_restore") end)
        end)
    else
        -- modo viejo: ves el TP
        if config.SoftStick then
            local cur = my.Position
            local goal = stickPos
            local delta = goal - cur
            local dist = delta.Magnitude
            local maxStep = math.max(config.SoftStickMaxStep or 8, 1)
            if dist > maxStep then
                goal = cur + delta.Unit * maxStep
            end
            my.CFrame = CFrame.lookAt(goal, lookPos)
        else
            my.CFrame = stickCF
        end
        if config.ForceZeroVelocity then
            pcall(function()
                my.AssemblyLinearVelocity = Vector3.zero
                my.AssemblyAngularVelocity = Vector3.zero
            end)
        end
    end

    -- disparar a head como rage
    if config.StickFireHead and modulesOK and FighterController then
        pcall(function()
            local lf = FighterController.LocalFighter
            local item = lf and lf.EquippedItem
            if item and head and fireRageUseItem then
                fireRageUseItem(item, stickCF, head, root)
            end
        end)
    end

    if config.CameraLookAtTarget and head and not config.CameraIndependent then
        LookAtPos(head.Position)
    end
    return true
end
function DoStrafe()
    -- SOLO si StrafeEnabled == true (nunca Orbit)
    if not config.StrafeEnabled then return end
    if config.OrbitEnabled then return end -- nunca mezclar
    if IsAntiUnnamedActive() or config.VoidSpamEnabled or config.RagebotEnabled or not CanDoDangerousMove() then return end
    local target = GetClosestEnemy()
    if not target or not target.Character or (config.TeamCheck and IsSameTeam(target)) then return end
    local root = target.Character:FindFirstChild("HumanoidRootPart")
    local head = target.Character:FindFirstChild("Head")
    if not root then return end
    local t = tick() * config.StrafeSpeed
    -- strafe lateral (no círculo completo)
    SafeEnemyCFrame(root, config.StickHeight, math.sin(t) * (config.OrbitRange or 4), 0, head and head.Position or root.Position)
end
function DoOrbit()
    -- SOLO si OrbitEnabled == true explícitamente
    if config.OrbitEnabled ~= true then return end
    if config.StrafeEnabled then return end
    if IsAntiUnnamedActive() or config.VoidSpamEnabled or config.RagebotEnabled or not CanDoDangerousMove() then return end
    local target = GetClosestEnemy()
    if not target or not target.Character or (config.TeamCheck and IsSameTeam(target)) then return end
    local root = target.Character:FindFirstChild("HumanoidRootPart")
    local head = target.Character:FindFirstChild("Head")
    if not root then return end
    OrbitAngle = OrbitAngle + (config.OrbitSpeed or 16) * 0.025
    SafeEnemyCFrame(root, config.StickHeight, math.cos(OrbitAngle) * (config.OrbitRange or 4), math.sin(OrbitAngle) * (config.OrbitRange or 4), head and head.Position or root.Position)
end
function ExpandHitboxes()
    if not config.HitboxExpand then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == LocalPlayer or not plr.Character then continue end
        if config.TeamCheck and IsSameTeam(plr) then continue end
        local root = plr.Character:FindFirstChild("HumanoidRootPart")
        if root and not root:FindFirstChild("SleepyHitbox") then
            local box = Instance.new("Part")
            box.Name = "SleepyHitbox"
            box.Size = Vector3.new(config.HitboxSize, config.HitboxSize, config.HitboxSize)
            box.Transparency = 1
            box.CanCollide = false
            box.Massless = true
            box.Parent = root
            local w = Instance.new("Weld")
            w.Part0, w.Part1, w.Parent = root, box, box
        end
    end
end

StartFly = function()
    SafeDisconnect("Fly")
    Connections.Fly = RunService.RenderStepped:Connect(function()
        if not config.FlyEnabled or IsAntiUnnamedActive() or config.VoidSpamEnabled then return end
        local hrp, hum, cam = getHRP(), getHum(), workspace.CurrentCamera
        if not hrp or not hum or not cam then return end
        EnsureCameraFree()
        hum.PlatformStand = true
        hrp.AssemblyAngularVelocity = Vector3.zero
        local speed = config.FlySpeed
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then speed = speed * config.FlyBoost end
        local move = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then move = move + cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then move = move - cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then move = move - cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then move = move + cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move = move + Vector3.new(0, 1, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then move = move + Vector3.new(0, -1, 0) end
        if move.Magnitude > 0 then
            hrp.AssemblyLinearVelocity = move.Unit * speed
            hrp.CFrame = hrp.CFrame + move.Unit * speed * 0.016
        else
            hrp.AssemblyLinearVelocity = Vector3.zero
        end
    end)
end
StopFly = function()
    SafeDisconnect("Fly")
    local hum = getHum()
    if hum then hum.PlatformStand = false end
end

StartStickRender = function()
    SafeDisconnect("StickRender")
    local stickFrame = 0
    Connections.StickRender = RunService.RenderStepped:Connect(function()
        stickFrame += 1
        if stickFrame % 2 ~= 0 then return end -- OPT
        if IsAntiUnnamedActive() or config.VoidSpamEnabled or config.RagebotEnabled then return end
        if not CanDoDangerousMove() or not config.StickOnRender then return end
        -- prioridad estricta: solo una a la vez, y solo si está activada
        if config.OrbitEnabled == true then
            DoOrbit()
            return
        end
        if config.StrafeEnabled == true then
            DoStrafe()
            return
        end
        if config.Enabled == true then
            if not CurrentTarget or not CurrentTarget.Character
                or not CurrentTarget.Character:FindFirstChildOfClass("Humanoid")
                or CurrentTarget.Character:FindFirstChildOfClass("Humanoid").Health <= 0
                or not IsEnemy(CurrentTarget) then
                CurrentTarget = GetClosestEnemy()
            end
            if CurrentTarget and IsEnemy(CurrentTarget) then
                StickToTarget(CurrentTarget)
            else
                CurrentTarget = nil
            end
        end
    end)
end
StartMainLoop = function()
    SafeDisconnect("Main")
    StartStickRender()
    local mainFrame = 0
    Connections.Main = RunService.Heartbeat:Connect(function()
        mainFrame += 1
        -- OPT: movement menos frecuente
        local needMove = config.SpeedEnabled or config.JumpPowerEnabled or config.SlideBoostEnabled
            or config.AntiFling or config.DesyncJitter
        if needMove then
            if mainFrame % 3 == 0 then UpdateMovement() end
        elseif mainFrame % 6 == 0 then
            UpdateMovement()
        end
        if not CanDoDangerousMove() then return end
        if IsAntiUnnamedActive() or config.VoidSpamEnabled then return end
        if mainFrame % 5 == 0 then ExpandHitboxes() end
        if config.StickOnRender and (config.Enabled or config.StrafeEnabled or config.OrbitEnabled) then
            ApplyFakePosition()
            return
        end
        if not config.Enabled or config.RagebotEnabled then return end
        if not CurrentTarget or not CurrentTarget.Character
            or not CurrentTarget.Character:FindFirstChildOfClass("Humanoid")
            or CurrentTarget.Character:FindFirstChildOfClass("Humanoid").Health <= 0
            or (config.TeamCheck and IsSameTeam(CurrentTarget)) then
            CurrentTarget = GetClosestEnemy()
        end
        if CurrentTarget and not (config.TeamCheck and IsSameTeam(CurrentTarget)) then
            StickToTarget(CurrentTarget)
        end
        ApplyFakePosition()
    end)
end





-- ==================== LUAHOOK ESP (integrado bien) ====================
do
    if type(State) ~= "table" then
        State = {}
    end
    State.ESPObjects = State.ESPObjects or {}
    State.PrimaryTarget = State.PrimaryTarget
    State.RainbowHue = State.RainbowHue or 0
end

local function getSafePlayers()
    local list = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p and p:IsA("Player") then
            list[#list + 1] = p
        end
    end
    return list
end

-- Config unificado (ESP + weather keys)
if type(Config) ~= "table" or getmetatable(Config) == nil then
    Config = setmetatable({}, {
        __index = function(_, k)
            if k == "ESP" then
                return config.ESPEnabled == true or config.ESP == true
            end
            if k == "Weather" then
                return config.WeatherEnabled == true or config.Weather == true
            end
            if k == "ThirdPersonEnabled" then
                return config.ThirdPerson == true or config.ThirdPersonEnabled == true
            end
            -- map UI names → LuaHook names
            if k == "ESPBox" then return config.ESPShowBox ~= false end
            if k == "ESPName" then return config.ESPShowName ~= false end
            if k == "ESPDistance" then return config.ESPShowDist ~= false end
            if k == "ESPHealth" or k == "ESPHealthBar" then return config.ESPShowHP ~= false end
            local v = config[k]
            if v ~= nil then return v end
            -- defaults sensatos para que se vea bien (estilo UE / LuaHook)
            local defaults = {
                ESPBoxStyle = "Corner Brackets",
                ESPBoxBrackets = true,
                ESPBoxThickness = 2,
                ESPCasingThickness = 2,
                ESPBoxColor = Color3.fromRGB(255, 255, 255),
                ESPBoxTransparency = 0,
                ESPBoxFill = false,
                ESPBoxScale = 1,
                ESPCornerLength = 0.28,
                ESPNameMode = "DisplayName",
                ESPTeamCheck = true,
                ESPMaxDistance = 100000,
                ESPDistanceScaling = true,
                ESPDistanceScalingRef = 50,
                ESPHealthColorMode = "Ramp",
                ESPHealthSmooth = true,
                ESPHealthNumberMode = "Always",
                ESPHealthGhost = true,
                ColorEnemy = Color3.fromRGB(255, 255, 255),
            }
            return defaults[k]
        end,
        __newindex = function(_, k, v)
            if k == "ESP" then
                config.ESPEnabled = v and true or false
                config.ESP = v and true or false
            elseif k == "Weather" then
                config.WeatherEnabled = v and true or false
                config.Weather = v and true or false
            else
                config[k] = v
            end
        end,
    })
end

local lp = LocalPlayer
Camera = workspace.CurrentCamera
pcall(function()
    workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
        Camera = workspace.CurrentCamera
    end)
end)

-- screenDraw opcional (LuaHook lo usa si existe Drawing)
if screenDraw == nil then
    screenDraw = nil
end


-- Helpers que el ESP de LuaHook necesita (estaban fuera del módulo original)
local function envIdOf(player)
    local id = nil
    pcall(function()
        if not player then return end
        id = player:GetAttribute("EnvironmentID")
        if id ~= nil then return end
        local f = nil
        pcall(function()
            local RS = game:GetService("ReplicatedStorage")
            -- best-effort: some fighters expose env via character attribute
            local char = player.Character
            if char then id = char:GetAttribute("EnvironmentID") end
        end)
    end)
    return id
end

local function isTeammate(player)
    if player == LocalPlayer or player == lp then return true end
    local myEnv, theirEnv = envIdOf(LocalPlayer), envIdOf(player)
    -- entornos distintos = no pelean (lobby / otra partida) → tratar como "no target"
    if myEnv ~= nil and theirEnv ~= nil and myEnv ~= theirEnv then return true end
    local a = LocalPlayer:GetAttribute("TeamID")
    local b = player:GetAttribute("TeamID")
    if a == nil or b == nil then
        if LocalPlayer.Team ~= nil and player.Team ~= nil then
            return LocalPlayer.Team == player.Team
        end
        return false
    end
    return a == b
end

local function isAlive(player)
    local c = player and player.Character
    local h = c and c:FindFirstChildOfClass("Humanoid")
    return h ~= nil and h.Health > 0
end

local function getHealth(player)
    local c = player and player.Character
    local h = c and c:FindFirstChildOfClass("Humanoid")
    if not h then return 0, 100 end
    return h.Health, math.max(h.MaxHealth, 1)
end

local function isVisible(worldPos)
    local cam = workspace.CurrentCamera
    if not cam or not worldPos then return true end
    local origin = cam.CFrame.Position
    local dir = worldPos - origin
    local dist = dir.Magnitude
    if dist < 1 then return true end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local ignore = { LocalPlayer.Character }
    pcall(function()
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character then table.insert(ignore, p.Character) end
        end
    end)
    params.FilterDescendantsInstances = ignore
    local hit = workspace:Raycast(origin, dir.Unit * dist, params)
    return hit == nil
end

local ESP = {}
;(function()
    local hasDrawing  = screenDraw ~= nil
    local _renderConn = nil
    local _espFrame   = 0
    local _lastRenderT = 0
    local _dcLastT    = 0
    local _bboxCache  = {}
    local _bboxFrameN = {}
    local _ctx        = {}
    local _renderArr  = {}
    local _dcArr      = {}
    local _radarO     = { drawings = {} }
    local _module     = { drawings = {} }
    local BLACK = Color3.new(0, 0, 0)
    local WHITE = Color3.new(1, 1, 1)
    local INK   = Color3.fromRGB(4, 6, 10)
    local HPBG  = Color3.fromRGB(11, 15, 22)
    local HP_W       = 5
    local HP_GAP     = 6
    local PAD        = 5
    local FLAG_GAP   = 6
    local BASE_H     = 1080
    local MIN_W      = 8
    local MIN_H      = 13
    local BOX_W_STUDS = 4   * 1000 / 1080
    local BOX_H_STUDS = 6.5 * 1000 / 1080
    local TEXT_FLOOR  = 8
    local DIST_MIN_MUL = 0.5
    local cam        = Camera
    local FACES = {}
    for _, n in ipairs({ "Code", "RobotoMono", "Gotham", "GothamBold", "Arial", "SourceSans" }) do
        local ok, f = pcall(function() return Enum.Font[n] end)
        if ok and f then FACES[n] = f end
    end
    if FACES.Code == nil then FACES.Code = Enum.Font.SourceSans end
    local _customFaces = {}
    local function faceFor(name)
        local c = _customFaces[name]
        if c ~= nil then return c end
        return FACES[name] or FACES.Code
    end
    local GLYPH_TRI = string.char(0xE2, 0x96, 0xB2)
    local GLYPH_MID = string.char(0xC2, 0xB7)
    local function typePx(base, scale)
        local v = math.floor(base * scale + 0.5)
        if v < TEXT_FLOOR then v = TEXT_FLOOR end
        return v
    end
    local function distMul(dist)
        if not Config.ESPDistanceScaling then return 1 end
        local ref = Config.ESPDistanceScalingRef or 50
        return math.clamp(ref / math.max(dist or 1, 1), DIST_MIN_MUL, 1)
    end
    local function sizeFor(base, mul)
        local v = math.floor(base * (mul or 1) + 0.5)
        if v < TEXT_FLOOR then v = TEXT_FLOOR end
        return v
    end
    local function styleLabel(t, face, size, casing)
        if typeof(face) == "EnumItem" then
            if t.Font ~= face then t.Font = face end
        else
            if t.FontFace ~= face then t.FontFace = face end
        end
        if t.TextSize ~= size then t.TextSize = size end
        local st = t:FindFirstChildOfClass("UIStroke")
        if st then
            local c = casing or 2
            if st.Thickness ~= c then st.Thickness = c end
            if st.Enabled ~= (c > 0) then st.Enabled = c > 0 end
        end
    end
    local _seqCache = {}
    local function gradSeq(key, a, b)
        if not a or not b then return nil end
        local s = _seqCache[key]
        if s and s.a == a and s.b == b then return s.seq end
        local seq = ColorSequence.new({
            ColorSequenceKeypoint.new(0, a),
            ColorSequenceKeypoint.new(0.5, b),
            ColorSequenceKeypoint.new(1, a),
        })
        _seqCache[key] = { a = a, b = b, seq = seq }
        return seq
    end
    local function paint(inst, prop, key, mode, solid, ga, gb, rot, spin, now)
        if inst == nil then return end
        if mode == "Gradient" then
            local seq = gradSeq(key, ga, gb)
            if seq then
                if inst[prop] ~= WHITE then inst[prop] = WHITE end
                local ug = inst:FindFirstChildOfClass("UIGradient")
                if ug == nil then ug = Instance.new("UIGradient"); ug.Parent = inst end
                if ug.Color ~= seq then ug.Color = seq end
                local r = rot or 0
                if spin and spin > 0 then r = (r + (now or 0) * spin * 360) % 360 end
                if ug.Rotation ~= r then ug.Rotation = r end
                if not ug.Enabled then ug.Enabled = true end
                return
            end
        end
        local ug = inst:FindFirstChildOfClass("UIGradient")
        if ug and ug.Enabled then ug.Enabled = false end
        local c = solid or WHITE
        if inst[prop] ~= c then inst[prop] = c end
    end
    local function healthColor(frac)
        local mode = Config.ESPHealthColorMode or "Ramp"
        if mode == "Solid" then return Config.ESPHealthColor or Color3.fromRGB(61, 224, 122) end
        if mode == "Gradient" then return WHITE end
        if hpRamp then return hpRamp(frac) end
        return Config.ESPHealthColor or Color3.fromRGB(61, 224, 122)
    end
    local _lock = { obj = nil, cand = nil, candT = 0 }
    local function resolveLock(bestO, bestD, lockO, lockD, now)
        local lk = _lock
        if bestO == nil then lk.obj = nil; lk.cand = nil; return nil end
        local prev = lk.obj
        if lk.obj == nil or lockO == nil then
            lk.obj = bestO; lk.cand = nil
        elseif bestO == lk.obj then
            lk.cand = nil
        elseif bestD <= (lockD or math.huge) * 0.9 then
            if lk.cand == bestO then
                if (now - (lk.candT or now)) >= 0.5 then lk.obj = bestO; lk.cand = nil end
            else
                lk.cand = bestO; lk.candT = now
            end
        else
            lk.cand = nil
        end
        if lk.obj and lk.obj ~= prev then lk.obj._primT = now end
        return lk.obj
    end
    local SKEL_R15 = {
        {"Head","UpperTorso"},{"UpperTorso","LowerTorso"},
        {"UpperTorso","LeftUpperArm"},{"LeftUpperArm","LeftLowerArm"},{"LeftLowerArm","LeftHand"},
        {"UpperTorso","RightUpperArm"},{"RightUpperArm","RightLowerArm"},{"RightLowerArm","RightHand"},
        {"LowerTorso","LeftUpperLeg"},{"LeftUpperLeg","LeftLowerLeg"},{"LeftLowerLeg","LeftFoot"},
        {"LowerTorso","RightUpperLeg"},{"RightUpperLeg","RightLowerLeg"},{"RightLowerLeg","RightFoot"},
    }
    local SKEL_R6 = {
        {"Head","Torso"},
        {"Torso","Left Arm"},{"Torso","Right Arm"},
        {"Torso","Left Leg"},{"Torso","Right Leg"},
    }
    local FLAG_COLORS = {
        STARING     = Color3.fromRGB(255,  70,  85),
        DEFLECT     = Color3.fromRGB( 53, 215, 199),
        SHIELD      = Color3.fromRGB(255, 194,  75),
        INVINCIBLE  = Color3.fromRGB(255, 215,   0),
        LOW         = Color3.fromRGB(255,  90, 100),
    }
    local _gui, _absLayer = nil, nil
    local function mkAbsLayer(g)
        local a = Instance.new("Frame")
        a.Name = "a"; a.BackgroundTransparency = 1; a.BorderSizePixel = 0
        a.Size = UDim2.fromScale(1, 1); a.Position = UDim2.new(); a.ZIndex = 1
        a.Parent = g
        return a
    end
    local function espGui()
        if _gui and _gui.Parent then
            if _absLayer == nil or _absLayer.Parent ~= _gui then _absLayer = mkAbsLayer(_gui) end
            return _gui
        end
        local ok, g = pcall(function()
            local s = Instance.new("ScreenGui")
            s.Name = "\0" .. tostring(math.random(1e5, 1e6))
            s.IgnoreGuiInset  = true
            s.ResetOnSpawn    = false
            s.DisplayOrder    = 99990
            s.ZIndexBehavior  = Enum.ZIndexBehavior.Sibling
            local parent
            pcall(function() parent = gethui and gethui() end)
            if parent == nil then pcall(function() parent = game:GetService("CoreGui") end) end
            if parent == nil then parent = lp:FindFirstChildOfClass("PlayerGui") end
            s.Parent = parent
            return s
        end)
        if not ok or not g then return nil end
        _gui = g
        _absLayer = mkAbsLayer(g)
        return _gui
    end
    local function mkFrame(parent, z)
        local f = Instance.new("Frame")
        f.BackgroundTransparency = 1
        f.BorderSizePixel = 0
        f.ZIndex = z or 1
        f.Parent = parent
        return f
    end
    local function mkLabel(parent, z, align)
        local t = Instance.new("TextLabel")
        t.BackgroundTransparency = 1
        t.BorderSizePixel = 0
        t.RichText = true
        t.TextXAlignment = align or Enum.TextXAlignment.Center
        t.AutomaticSize = Enum.AutomaticSize.XY
        t.Size = UDim2.fromOffset(0, 0)
        t.ZIndex = z or 3
        t.TextColor3 = WHITE
        local st = Instance.new("UIStroke")
        st.Color = BLACK; st.Thickness = 1; st.Transparency = 0
        st.LineJoinMode = Enum.LineJoinMode.Round
        st.Parent = t
        t.Parent = parent
        return t
    end
    local function mkList(parent, z, pad, horizAlign, vertAlign)
        local f = mkFrame(parent, z)
        f.AutomaticSize = Enum.AutomaticSize.XY
        f.Size = UDim2.fromOffset(0, 0)
        local l = Instance.new("UIListLayout")
        l.FillDirection        = Enum.FillDirection.Vertical
        l.SortOrder            = Enum.SortOrder.LayoutOrder
        l.Padding              = UDim.new(0, pad or 2)
        l.HorizontalAlignment  = horizAlign or Enum.HorizontalAlignment.Center
        l.VerticalAlignment    = vertAlign or Enum.VerticalAlignment.Top
        l.Parent = f
        return f
    end
    local function mkRing(parent, z, colour, thick)
        local f = mkFrame(parent, z)
        f.AnchorPoint = Vector2.new(0.5, 0.5)
        f.Position = UDim2.fromScale(0.5, 0.5)
        f.Size = UDim2.fromScale(1, 1)
        local s = Instance.new("UIStroke")
        s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        s.LineJoinMode    = Enum.LineJoinMode.Miter
        s.Color = colour; s.Thickness = thick; s.Transparency = 0
        s.Parent = f
        return f, s
    end
    local function mkCasedRect(parent, z)
        local box = mkFrame(parent, z)
        box.Size = UDim2.fromScale(1, 1)
        -- outline: outer black, mid color, inner black (más legible tipo UE)
        local rIn,  sIn  = mkRing(box, z + 1, INK,   2)
        local rMid, sMid = mkRing(box, z + 2, WHITE, 2)
        local rOut, sOut = mkRing(box, z + 1, INK,   2)
        return box, sMid, sOut, sIn, rMid, rOut, rIn
    end
    local function buildTree(o)
        local g = espGui(); if not g then return nil end
        local u = {}
        local root = mkFrame(g, 2)
        root.Name = "r"
        root.Visible = false
        root.Size = UDim2.fromOffset(MIN_W, MIN_H)
        u.root = root
        u.fill = mkFrame(root, 2)
        u.fill.Size = UDim2.fromScale(1, 1)
        u.fill.Visible = false
        u.box, u.boxStroke, u.caseOut, u.caseIn, u.ringMid, u.ringOut, u.ringIn = mkCasedRect(root, 3)
        u.box.Visible = false
        u.corners = {}
        for i = 1, 8 do
            local c = mkFrame(root, 6)
            c.BackgroundTransparency = 0
            c.BackgroundColor3 = WHITE
            c.Visible = false
            local cs = Instance.new("UIStroke")
            cs.Color = INK; cs.Thickness = 1; cs.Transparency = 0
            cs.LineJoinMode = Enum.LineJoinMode.Miter
            cs.Parent = c
            u.corners[i] = c
        end
        local hp = mkFrame(root, 4)
        hp.AnchorPoint = Vector2.new(1, 0)
        hp.Position = UDim2.new(0, -HP_GAP, 0, 0)
        hp.Size = UDim2.new(0, HP_W, 1, 0)
        hp.BackgroundTransparency = 0
        hp.BackgroundColor3 = HPBG
        local hs = Instance.new("UIStroke")
        hs.Color = INK; hs.Thickness = 1; hs.Transparency = 0
        hs.LineJoinMode = Enum.LineJoinMode.Miter
        hs.Parent = hp
        hp.Visible = false
        u.hp = hp
        u.hpGhost = mkFrame(hp, 5)
        u.hpGhost.BackgroundTransparency = 0
        u.hpGhost.BackgroundColor3 = WHITE
        u.hpGhost.AnchorPoint = Vector2.new(0, 1)
        u.hpGhost.Position = UDim2.fromScale(0, 1)
        u.hpGhost.Visible = false
        u.hpFill = mkFrame(hp, 6)
        u.hpFill.BackgroundTransparency = 0
        u.hpFill.BackgroundColor3 = WHITE
        u.hpFill.AnchorPoint = Vector2.new(0, 1)
        u.hpFill.Position = UDim2.fromScale(0, 1)
        u.hpFill.Size = UDim2.fromScale(1, 1)
        u.hpNum = mkLabel(root, 7, Enum.TextXAlignment.Right)
        u.hpNum.AnchorPoint = Vector2.new(1, 0)
        u.hpNum.Position = UDim2.new(0, -HP_GAP - HP_W - 3, 0, 0)
        u.hpNum.Visible = false
        local head = mkList(root, 7, 2, Enum.HorizontalAlignment.Center, Enum.VerticalAlignment.Bottom)
        head.AnchorPoint = Vector2.new(0.5, 1)
        head.Position = UDim2.new(0.5, 0, 0, -PAD)
        head.Visible = false
        u.head = head
        u.name = mkLabel(head, 8)
        u.name.LayoutOrder = 1
        u.under = mkFrame(head, 8)
        u.under.LayoutOrder = 2
        u.under.BackgroundTransparency = 0
        u.under.BackgroundColor3 = WHITE
        u.under.Size = UDim2.fromOffset(0, 2)
        u.under.Visible = false
        local foot = mkList(root, 7, 2, Enum.HorizontalAlignment.Center, Enum.VerticalAlignment.Top)
        foot.AnchorPoint = Vector2.new(0.5, 0)
        foot.Position = UDim2.new(0.5, 0, 1, PAD)
        foot.Visible = false
        u.foot = foot
        u.info = mkLabel(foot, 8)
        u.info.LayoutOrder = 1
        u.info.Visible = false
        local ammo = mkFrame(foot, 8)
        ammo.LayoutOrder = 2
        ammo.BackgroundTransparency = 0
        ammo.BackgroundColor3 = HPBG
        ammo.Size = UDim2.fromOffset(28, 3)
        local as = Instance.new("UIStroke")
        as.Color = INK; as.Thickness = 1; as.Transparency = 0
        as.LineJoinMode = Enum.LineJoinMode.Miter
        as.Parent = ammo
        ammo.Visible = false
        u.ammo = ammo
        u.ammoFill = mkFrame(ammo, 9)
        u.ammoFill.BackgroundTransparency = 0
        u.ammoFill.BackgroundColor3 = WHITE
        u.ammoFill.Size = UDim2.fromScale(1, 1)
        local flags = mkList(root, 7, 3, Enum.HorizontalAlignment.Left, Enum.VerticalAlignment.Top)
        flags.AnchorPoint = Vector2.new(0, 0)
        flags.Position = UDim2.new(1, FLAG_GAP, 0, 0)
        flags.Visible = false
        u.flags = flags
        u.chips = {}
        for i = 1, 5 do
            local c = mkLabel(flags, 8, Enum.TextXAlignment.Left)
            c.LayoutOrder = i
            c.Visible = false
            u.chips[i] = c
        end
        u.chev = mkLabel(root, 9)
        u.chev.AnchorPoint = Vector2.new(0.5, 1)
        u.chev.Text = GLYPH_TRI
        u.chev.Visible = false
        u.pulse = mkFrame(root, 1)
        u.pulse.AnchorPoint = Vector2.new(0.5, 0.5)
        u.pulse.Position = UDim2.fromScale(0.5, 0.5)
        local ps = Instance.new("UIStroke")
        ps.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        ps.LineJoinMode = Enum.LineJoinMode.Miter
        ps.Thickness = 2; ps.Color = WHITE
        ps.Parent = u.pulse
        u.pulseStroke = ps
        u.pulse.Visible = false
        local a = _absLayer
        u.skel = {}
        u.tracer  = mkFrame(a, 2); u.tracer.AnchorPoint = Vector2.new(0.5, 0.5)
        u.tracer.BackgroundTransparency = 0; u.tracer.BackgroundColor3 = WHITE; u.tracer.Visible = false
        u.look    = mkFrame(a, 2); u.look.AnchorPoint = Vector2.new(0.5, 0.5)
        u.look.BackgroundTransparency = 0; u.look.BackgroundColor3 = WHITE; u.look.Visible = false
        u.dot     = mkFrame(a, 3); u.dot.AnchorPoint = Vector2.new(0.5, 0.5)
        u.dot.BackgroundTransparency = 0; u.dot.BackgroundColor3 = WHITE; u.dot.Visible = false
        local dc = Instance.new("UICorner"); dc.CornerRadius = UDim.new(1, 0); dc.Parent = u.dot
        local ds = Instance.new("UIStroke"); ds.Color = INK; ds.Thickness = 1; ds.Parent = u.dot
        return u
    end
    local function destroyTree(o)
        local u = o.ui; if not u then return end
        pcall(function() if u.root then u.root:Destroy() end end)
        pcall(function() if u.tracer then u.tracer:Destroy() end end)
        pcall(function() if u.look then u.look:Destroy() end end)
        pcall(function() if u.dot then u.dot:Destroy() end end)
        if u.skel then
            for i = 1, #u.skel do pcall(function() u.skel[i]:Destroy() end) end
        end
        o.ui = nil
    end
    local _createBudget = 0
    local CREATE_BUDGET_PER_FRAME = 6 -- OPT lag
    local function newDraw(t)
        if not hasDrawing then return nil end
        local ok, d = pcall(screenDraw, t)
        if not ok then return nil end
        d.Visible = false; return d
    end
    local function nd(o, key, dtype)
        local d = o[key]
        if d == nil then
            if _createBudget <= 0 then return nil end
            _createBudget = _createBudget - 1
            d = newDraw(dtype) or false
            o[key] = d
            if d then o.drawings[#o.drawings + 1] = d end
        end
        return d or nil
    end
    local function ndArr(o, key, dtype, n)
        local arr = o[key]
        if not arr then arr = {}; o[key] = arr end
        for i = #arr + 1, n do
            if _createBudget <= 0 then break end
            local d = newDraw(dtype)
            if not d then break end
            _createBudget = _createBudget - 1
            arr[i] = d
            o.drawings[#o.drawings + 1] = d
        end
        return arr
    end
    local function ensureCham(o, char)
        local hl = o.cham
        if hl == nil then
            hl = Instance.new("Highlight")
            hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            hl.Enabled   = false
            hl.Parent    = char
            o.cham = hl
        elseif hl and hl.Parent ~= char then
            pcall(function() hl.Parent = char end)
        end
        return o.cham
    end
    local function cleanESP(p)
        local o = State.ESPObjects[p]; if not o then return end
        destroyTree(o)
        for _, d in ipairs(o.drawings or {}) do
            pcall(function() if d.Remove then d:Remove() end end)
        end
        if o.cham and o.cham.Parent then pcall(function() o.cham:Destroy() end) end
        _bboxCache[p]  = nil
        _bboxFrameN[p] = nil
        State.ESPObjects[p] = nil
    end
    local function buildESP(player)
        if player == lp then return end
        cleanESP(player)
        local char = player.Character; if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
        local hum  = char:FindFirstChildOfClass("Humanoid")
        local isR6 = (hum and hum.RigType == Enum.HumanoidRigType.R6) or (char:FindFirstChild("Torso") ~= nil)
        local rig  = isR6 and SKEL_R6 or SKEL_R15
        local bones = {}
        for i, pair in ipairs(rig) do
            bones[i] = { a = char:FindFirstChild(pair[1]), b = char:FindFirstChild(pair[2]) }
        end
        local o = { drawings = {}, root = root, rig = rig, bones = bones, _fadeT = tick() }
        o.ui = buildTree(o)
        State.ESPObjects[player] = o
    end
    local function bbox(char, player)
        local frame = _bboxFrameN[player] or -999
        if (_espFrame - frame) < 1 then
            local c = _bboxCache[player]
            if c then return c[1], c[2], c[3], c[4] end
            return nil
        end
        _bboxFrameN[player] = _espFrame
        _bboxCache[player] = nil
        local ok, pivot = pcall(function() return char:GetPivot().Position end)
        if not ok or pivot == nil then return nil end
        local sp = cam:WorldToViewportPoint(pivot)
        local depth = sp.Z
        if depth <= 0.5 then return nil end
        local vpY = _ctx.vp and _ctx.vp.Y or 1080
        local tanHalf = math.tan(math.rad(cam.FieldOfView) * 0.5)
        if tanHalf <= 0 then return nil end
        local scale = vpY / (2 * depth * tanHalf)
        local bs = Config.ESPBoxScale or 1
        local rawW = BOX_W_STUDS * scale * bs
        local rawH = BOX_H_STUDS * scale * bs
        if rawH > 4000 or rawW > 3000 then return nil end
        local x = math.floor(sp.X - rawW * 0.5)
        local y = math.floor(sp.Y - rawH * 0.5)
        local bw = math.floor(math.max(MIN_W, rawW))
        local bh = math.floor(math.max(MIN_H, rawH))
        _bboxCache[player] = { x, y, x + bw, y + bh }
        return x, y, x + bw, y + bh
    end
    local function hideTree(o)
        local u = o.ui; if not u then return end
        if u.root then u.root.Visible = false end
        if u.tracer then u.tracer.Visible = false end
        if u.look then u.look.Visible = false end
        if u.dot then u.dot.Visible = false end
        if u.skel then for i = 1, #u.skel do u.skel[i].Visible = false end end
    end
    local function hideDrawings(o)
        for _, d in ipairs(o.drawings) do d.Visible = false end
    end
    local function hideAll(o, force)
        if o._allHidden and not force then return end
        o._allHidden = true
        hideTree(o)
        hideDrawings(o)
        if o.cham then o.cham.Enabled = false end
    end
    function ESP.rebuildAll()
        for p in pairs(State.ESPObjects) do cleanESP(p) end
        if not Config.ESP then return end
        for _, p in ipairs(getSafePlayers()) do
            if p ~= lp and p.Character then buildESP(p) end
        end
    end
    function ESP.registerFace(name, font)
        if type(name) == "string" and font ~= nil then _customFaces[name] = font end
    end
    local function espWeapon(o, player)
        local now = tick()
        if not o._wepT or (now - o._wepT) > 0.5 then
            o._wep  = getWeaponName(player)
            o._wepT = now
        end
        return o._wep or "?"
    end
    local function paintBox(u, ctx)
        local mode = Config.ESPBoxColorMode or "Solid"
        paint(u.boxStroke, "Color", "box", mode, Config.ESPBoxColor, Config.ESPBoxGradA, Config.ESPBoxGradB,
              Config.ESPGradientRotBox or 0, Config.ESPGradientSpeed, ctx.now)
        -- outline negro más grueso + stroke de color
        local ct = math.floor(math.clamp(Config.ESPCasingThickness or 2, 1, 4))
        local bt = math.floor(math.clamp(Config.ESPBoxThickness or 2, 1, 5))
        if ctx.primary and Config.ESPPrimaryEmphasis then bt = bt + 1 end
        if u.caseOut then
            u.caseOut.Thickness = ct
            u.caseOut.Color = Color3.fromRGB(0, 0, 0)
        end
        if u.caseIn then
            u.caseIn.Thickness = ct
            u.caseIn.Color = Color3.fromRGB(0, 0, 0)
        end
        if u.boxStroke.Thickness ~= bt then u.boxStroke.Thickness = bt end
        local sOut = UDim2.fromScale(1, 1)
        local sMid = UDim2.new(1, -2 * bt, 1, -2 * bt)
        local sIn  = UDim2.new(1, -2 * bt - 2 * ct, 1, -2 * bt - 2 * ct)
        if u.ringOut.Size ~= sOut then u.ringOut.Size = sOut end
        if u.ringMid.Size ~= sMid then u.ringMid.Size = sMid end
        if u.ringIn.Size  ~= sIn  then u.ringIn.Size  = sIn  end
        local tr = math.clamp((Config.ESPBoxTransparency or 0) + (1 - (ctx.fade or 1)), 0, 1)
        if u.boxStroke.Transparency ~= tr then u.boxStroke.Transparency = tr end
        if u.caseOut.Transparency ~= tr then u.caseOut.Transparency = tr end
        if u.caseIn.Transparency  ~= tr then u.caseIn.Transparency  = tr end
    end
    local function drawCorners(u, w, h, ctx)
        local len = math.floor(math.clamp(w * (Config.ESPCornerLength or 0.28), 4, math.max(w * 0.48, 6)))
        local th  = math.max(math.floor(Config.ESPBoxThickness or 1), 1)
        local col = Config.ESPBoxColor or WHITE
        local mode = Config.ESPBoxColorMode or "Solid"
        local spec = {
            { 0, 0, len, th }, { 0, 0, th, len },
            { w - len, 0, len, th }, { w - th, 0, th, len },
            { 0, h - th, len, th }, { 0, h - len, th, len },
            { w - len, h - th, len, th }, { w - th, h - len, th, len },
        }
        for i = 1, 8 do
            local c = u.corners[i]
            local s = spec[i]
            c.Position = UDim2.fromOffset(s[1], s[2])
            c.Size     = UDim2.fromOffset(s[3], s[4])
            paint(c, "BackgroundColor3", "box", mode, col, Config.ESPBoxGradA, Config.ESPBoxGradB,
                  Config.ESPGradientRotBox or 0, Config.ESPGradientSpeed, ctx.now)
            local cs = c:FindFirstChildOfClass("UIStroke")
            if cs then
                cs.Color = Color3.fromRGB(0, 0, 0)
                cs.Thickness = math.max(math.floor(Config.ESPCasingThickness or 2), 1)
                cs.Transparency = 0
            end
            c.Visible = true
        end
    end
    local function drawHealth(u, o, ctx, frac, hp)
        if not Config.ESPHealth then u.hp.Visible = false; u.hpNum.Visible = false; return end
        local fill = frac
        if Config.ESPHealthSmooth ~= false then
            local s = o._sHP
            if s == nil then s = frac else s = s + (frac - s) * math.clamp((ctx.dt or 0) * 14, 0, 1) end
            o._sHP = s; fill = s
        end
        fill = math.clamp(fill, 0, 1)
        u.hp.Visible = true
        -- fondo más oscuro + borde negro fuerte (outline)
        u.hp.BackgroundColor3 = Color3.fromRGB(8, 10, 14)
        u.hp.BackgroundTransparency = 0.15
        local hs = u.hp:FindFirstChildOfClass("UIStroke")
        if hs then
            hs.Color = Color3.fromRGB(0, 0, 0)
            hs.Thickness = 2
            hs.Transparency = 0
        end
        -- fill con padding interno (más limpio)
        local pad = 1
        u.hpFill.Position = UDim2.new(0, pad, 1, -pad)
        u.hpFill.AnchorPoint = Vector2.new(0, 1)
        u.hpFill.Size = UDim2.new(1, -pad * 2, fill, -pad)
        -- color ramp mejorado: verde vivo → amarillo → rojo
        local col
        if fill > 0.6 then
            local t = (fill - 0.6) / 0.4
            col = Color3.fromRGB(
                math.floor(80 + (255 - 80) * (1 - t)),
                255,
                math.floor(80 * (1 - t))
            )
        elseif fill > 0.3 then
            local t = (fill - 0.3) / 0.3
            col = Color3.fromRGB(255, math.floor(200 * t + 55 * (1 - t)), 40)
        else
            local t = fill / 0.3
            col = Color3.fromRGB(255, math.floor(55 * t), 40)
        end
        if Config.ESPHealthColorMode == "Solid" then
            col = Config.ESPHealthColor or Color3.fromRGB(80, 255, 120)
        end
        u.hpFill.BackgroundColor3 = col
        u.hpFill.BackgroundTransparency = Config.ESPHealthTransparency or 0
        -- ghost damage (hit reciente)
        if Config.ESPHealthGhost and o._ghostT and o._ghostFrac and o._ghostFrac > fill
                and (ctx.now - o._ghostT) < 0.4 then
            u.hpGhost.Visible = true
            u.hpGhost.Position = UDim2.new(0, pad, 1, -pad)
            u.hpGhost.AnchorPoint = Vector2.new(0, 1)
            u.hpGhost.Size = UDim2.new(1, -pad * 2, math.clamp(o._ghostFrac, 0, 1), -pad)
            u.hpGhost.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            u.hpGhost.BackgroundTransparency = 0.4 + 0.55 * math.clamp((ctx.now - o._ghostT) / 0.4, 0, 1)
        else
            u.hpGhost.Visible = false
        end
        -- número de HP siempre legible
        local nm = Config.ESPHealthNumberMode or "Always"
        local show = nm == "Always" or (nm == "OnDamage" and frac < 0.999)
        if show then
            u.hpNum.Visible = true
            u.hpNum.Text = tostring(math.floor(hp or 0))
            styleLabel(u.hpNum, ctx.face, sizeFor((ctx.tsHp or 12) + 1, ctx.dmul), ctx.casing)
            u.hpNum.TextColor3 = col
            u.hpNum.TextTransparency = 1 - (ctx.fade or 1)
            u.hpNum.Position = UDim2.new(0, -HP_GAP - HP_W - 4, math.clamp(1 - fill, 0, 1), 0)
        else
            u.hpNum.Visible = false
        end
    end
    local function drawText(u, o, player, ctx, dist, frac)
        local declut = Config.ESPDeclutter and o._declutter
        if Config.ESPName and not declut then
            u.head.Visible = true
            u.name.Visible = true
            styleLabel(u.name, ctx.face, sizeFor(ctx.tsName, ctx.dmul), ctx.casing)
            u.name.Text = (Config.ESPNameMode == "Username") and player.Name or player.DisplayName
            paint(u.name, "TextColor3", "name", Config.ESPNameColorMode or "Solid", Config.ESPNameColor,
                  Config.ESPNameGradA, Config.ESPNameGradB, Config.ESPGradientRotText or 90,
                  Config.ESPGradientSpeed, ctx.now)
            u.name.TextTransparency = math.clamp((Config.ESPNameTransparency or 0) + (1 - (ctx.fade or 1)), 0, 1)
            if Config.ESPNameHealthUnderline and frac then
                local tb = u.name.TextBounds
                u.under.Visible = true
                u.under.Size = UDim2.fromOffset(math.max(math.floor((tb and tb.X or 0) * frac), 1), 2)
                u.under.BackgroundColor3 = healthColor(frac)
            else
                u.under.Visible = false
            end
        else
            u.head.Visible = false
        end
        local showInfo = (Config.ESPDistance or Config.ESPWeapon) and not declut
        local showAmmo = Config.ESPAmmoBar and not declut
        u.foot.Visible = showInfo or showAmmo
        if showInfo then
            local txt
            if Config.ESPWeapon then txt = espWeapon(o, player) end
            if Config.ESPDistance then
                local d = ("%dm"):format(dist)
                txt = txt and (txt .. " " .. GLYPH_MID .. " " .. d) or d
            end
            u.info.Visible = true
            styleLabel(u.info, ctx.face, sizeFor(ctx.tsInfo, ctx.dmul), ctx.casing)
            u.info.Text = txt or ""
            paint(u.info, "TextColor3", "info", Config.ESPInfoColorMode or "Solid", Config.ESPInfoColor,
                  Config.ESPInfoGradA, Config.ESPInfoGradB, Config.ESPGradientRotText or 90,
                  Config.ESPGradientSpeed, ctx.now)
            u.info.TextTransparency = 1 - (ctx.fade or 1)
        else
            u.info.Visible = false
        end
        if showAmmo then
            local ammo, maxAmmo = 30, 30
            pcall(function()
                local item = Rivals.Fighter and Rivals.Fighter:Get(player)
                local eq = item and (item.EquippedItem or (item.GetEquippedItem and item:GetEquippedItem()))
                if eq and eq.Data then
                    ammo = eq.Data.Ammo or 30
                    maxAmmo = eq.Data.MaxAmmo or 30
                end
            end)
            local pct = math.clamp(maxAmmo > 0 and (ammo / maxAmmo) or 1, 0, 1)
            u.ammo.Visible = true
            u.ammo.Size = UDim2.fromOffset(math.max(math.floor(ctx.boxW * 0.9), 12), 3)
            u.ammoFill.Size = UDim2.new(pct, 0, 1, 0)
            paint(u.ammoFill, "BackgroundColor3", "mark", Config.ESPMarkColorMode or "Solid",
                  Config.ESPMarkColor, Config.ESPMarkGradA, Config.ESPMarkGradB, 0,
                  Config.ESPGradientSpeed, ctx.now)
        else
            u.ammo.Visible = false
        end
    end
    local function drawFlags(u, o, player, ctx, char, frac)
        local n = 0
        local function add(key, label)
            if n >= 5 then return end
            n = n + 1
            local c = u.chips[n]
            local mode = Config.ESPFlagColorMode or "PerFlag"
            c.Visible = true
            styleLabel(c, ctx.face, sizeFor(ctx.tsChip, ctx.dmul), ctx.casing)
            c.Text = label
            paint(c, "TextColor3", "flag", mode,
                  (mode == "PerFlag") and (FLAG_COLORS[key] or WHITE) or Config.ESPFlagColor,
                  Config.ESPFlagGradA, Config.ESPFlagGradB, Config.ESPGradientRotText or 90,
                  Config.ESPGradientSpeed, ctx.now)
            c.TextTransparency = 1 - (ctx.fade or 1)
        end
        if Config.ESPFlagStaring then
            local head = char:FindFirstChild("Head")
            local myPos = ctx.myRoot and ctx.myRoot.Position
            if head and myPos then
                local toMe = myPos - head.Position
                if toMe.Magnitude > 1 then
                    toMe = toMe.Unit
                    local look = head.CFrame.LookVector
                    if (look.X * toMe.X + look.Y * toMe.Y + look.Z * toMe.Z) > 0.965 then add("STARING", "[STARING]") end
                end
            end
        end
        if Config.ESPFlagDeflect and isDeflecting and isDeflecting(player) then add("DEFLECT", "[DEFLECT]") end
        if Config.ESPFlagShield then
            local wep = espWeapon(o, player)
            if wep and (wep:find("Shield") or wep:find("Riot")) then add("SHIELD", "[SHIELD]") end
        end
        if Config.ESPFlagInvincible and isSpawnProtected and isSpawnProtected(player) then add("INVINCIBLE", "[INVINCIBLE]") end
        if Config.ESPFlagLowHP and frac and frac <= 0.25 then add("LOW", "[LOW HP]") end
        u.flags.Visible = n > 0
        for i = n + 1, 5 do u.chips[i].Visible = false end
    end
    local function segment(f, ax, ay, bx, by, thick, transp)
        local dx, dy = bx - ax, by - ay
        local len = math.sqrt(dx * dx + dy * dy)
        if len < 1 then f.Visible = false; return end
        f.Position = UDim2.fromOffset(math.floor((ax + bx) * 0.5), math.floor((ay + by) * 0.5))
        f.Size = UDim2.fromOffset(math.floor(len + 0.5), thick)
        f.Rotation = math.deg(math.atan2(dy, dx))
        f.BackgroundTransparency = transp or 0
        f.Visible = true
    end
    local function drawSkeleton(u, o, ctx)
        if not Config.ESPSkeleton then
            if u.skel then for i = 1, #u.skel do u.skel[i].Visible = false end end
            return
        end
        local nb = #o.bones
        for i = #u.skel + 1, nb do
            local f = mkFrame(_absLayer, 2)
            f.AnchorPoint = Vector2.new(0.5, 0.5)
            f.BackgroundTransparency = 0
            f.BackgroundColor3 = WHITE
            local s = Instance.new("UIStroke")
            s.Color = INK; s.Thickness = 1; s.Transparency = 0
            s.Parent = f
            f.Visible = false
            u.skel[i] = f
        end
        if (not o._skelPts) or (_espFrame - (o._skelFrame or -9)) >= 4 then
            o._skelFrame = _espFrame
            local pts = o._skelPts; if not pts then pts = {}; o._skelPts = pts end
            for i, pair in ipairs(o.bones) do
                local pa, pb = pair.a, pair.b
                local e = pts[i]; if not e then e = {}; pts[i] = e end
                if pa and pb and pa.Parent and pb.Parent then
                    local s1, v1 = cam:WorldToViewportPoint(pa.Position)
                    local s2, v2 = cam:WorldToViewportPoint(pb.Position)
                    if v1 and v2 and s1.Z > 0 and s2.Z > 0 then
                        e.ok = true; e.ax = s1.X; e.ay = s1.Y; e.bx = s2.X; e.by = s2.Y
                    else e.ok = false end
                else e.ok = false end
            end
        end
        local th = math.max(math.floor(Config.ESPSkeletonThickness or 1), 1)
        local mode = Config.ESPSkeletonColorMode or "Solid"
        local pts = o._skelPts
        for i = 1, nb do
            local f = u.skel[i]
            local e = pts and pts[i]
            if f then
                if e and e.ok then
                    segment(f, e.ax, e.ay, e.bx, e.by, th, Config.ESPSkeletonTransparency or 0)
                    paint(f, "BackgroundColor3", "skel", mode, Config.ESPSkeletonColor,
                          Config.ESPSkeletonGradA, Config.ESPSkeletonGradB, 0, Config.ESPGradientSpeed, ctx.now)
                else f.Visible = false end
            end
        end
    end
    local function drawAbsExtras(u, o, ctx, char, minX, minY, maxX, maxY)
        if Config.ESPTracers then
            local vp = ctx.vp
            local ox, oy = vp.X * 0.5, vp.Y
            local m = Config.ESPTracerOrigin
            if m == "Top" then oy = 0
            elseif m == "Middle" then oy = vp.Y * 0.5
            elseif m == "Mouse" then
                local mp; pcall(function() mp = UserInputService:GetMouseLocation() end)
                if mp then ox, oy = mp.X, mp.Y end
            end
            segment(u.tracer, ox, oy, (minX + maxX) * 0.5, maxY,
                    math.max(math.floor(Config.ESPTracerThickness or 1), 1), Config.ESPTracerTransparency or 0)
            paint(u.tracer, "BackgroundColor3", "tracer", Config.ESPTracerColorMode or "Solid",
                  Config.ESPTracerColor, Config.ESPTracerGradA, Config.ESPTracerGradB, 0,
                  Config.ESPGradientSpeed, ctx.now)
        else u.tracer.Visible = false end
        if Config.ESPHeadDot then
            local head = char:FindFirstChild("Head")
            local ok = false
            if head then
                local hsp, inv = cam:WorldToViewportPoint(head.Position)
                if inv and hsp.Z > 0 then
                    local r = math.clamp(Config.ESPHeadDotSize or 4, 2, 16)
                    u.dot.Position = UDim2.fromOffset(math.floor(hsp.X), math.floor(hsp.Y))
                    u.dot.Size = UDim2.fromOffset(r * 2, r * 2)
                    paint(u.dot, "BackgroundColor3", "mark", Config.ESPMarkColorMode or "Solid",
                          Config.ESPMarkColor, Config.ESPMarkGradA, Config.ESPMarkGradB, 0,
                          Config.ESPGradientSpeed, ctx.now)
                    u.dot.Visible = true; ok = true
                end
            end
            if not ok then u.dot.Visible = false end
        else u.dot.Visible = false end
        if Config.ESPLookLine then
            local head = char:FindFirstChild("Head")
            local ok = false
            if head then
                local look = head.CFrame.LookVector
                local p1 = head.Position
                local p2 = p1 + look * math.clamp(Config.ESPLookLineLength or 8, 2, 32)
                local s1, v1 = cam:WorldToViewportPoint(p1)
                local s2, v2 = cam:WorldToViewportPoint(p2)
                if v1 and v2 and s1.Z > 0 and s2.Z > 0 then
                    segment(u.look, s1.X, s1.Y, s2.X, s2.Y, 1, 0)
                    paint(u.look, "BackgroundColor3", "mark", Config.ESPMarkColorMode or "Solid",
                          Config.ESPMarkColor, Config.ESPMarkGradA, Config.ESPMarkGradB, 0,
                          Config.ESPGradientSpeed, ctx.now)
                    ok = true
                end
            end
            if not ok then u.look.Visible = false end
        else u.look.Visible = false end
    end
    local function renderPlayer(player, o)
        local ctx  = _ctx
        local char = player.Character
        if not char or not o.root or not o.root.Parent then cleanESP(player); return end
        if not o.root:IsA("BasePart") then cleanESP(player); return end
        local rootPos = o.root.Position
        if not rootPos then cleanESP(player); return end
        if not o.ui then o.ui = buildTree(o); if not o.ui then return end end
        local u = o.ui
        local vp     = ctx.vp
        local myRoot = ctx.myRoot
        local dist   = o._dist
        if not dist then
            dist = (myRoot and myRoot.Position and rootPos)
                and (myRoot.Position - rootPos).Magnitude or 0
        end
        local oor = dist > Config.ESPMaxDistance
        if (Config.ESPTeamCheck and isTeammate(player)) or (not isAlive(player)) then
            hideAll(o); return
        end
        if (Config.ESPRadar or Config.ESPChamsVisSplit or Config.ESPPeekAlert
                or (Config.ESPChams and Config.ESPChamsStyle == "Ghost"))
            and not oor and (_espFrame - (o._visFrame or -99)) >= 3 then
            pcall(function() o._vis = isVisible(rootPos) end)
            o._visFrame = _espFrame
        end
        if oor then
            if o.cham then o.cham.Enabled = false end
            hideTree(o); hideDrawings(o); o._allHidden = true
            return
        end
        if o._allHidden and Config.ESPFadeIn then o._fadeT = ctx.now end
        o._allHidden = false
        local fade = 1
        if Config.ESPFadeIn and o._fadeT then
            fade = math.clamp((ctx.now - o._fadeT) / 0.12, 0, 1)
        end
        ctx.fade = fade
        ctx.primary = o._primary and true or false
        ctx.dmul = distMul(dist)
        local minX, minY, maxX, maxY = bbox(char, player)
        if not minX then
            hideTree(o)
        else
            local w, h = maxX - minX, maxY - minY
            ctx.boxW = w
            u.root.Position = UDim2.fromOffset(minX, minY)
            u.root.Size     = UDim2.fromOffset(w, h)
            u.root.Visible  = true
            local hp, mh, frac
            if Config.ESPHealth or Config.ESPHealthGhost or Config.ESPFlagLowHP or Config.ESPNameHealthUnderline then
                pcall(function() hp, mh = getHealth(player) end)
                if hp ~= nil then frac = math.clamp((mh or 0) > 0 and hp / mh or 0, 0, 1) end
            end
            local prevHP = o._lastHP
            if Config.ESPHealthGhost and prevHP and frac and frac < prevHP - 0.005 then
                if not o._ghostT or (ctx.now - o._ghostT) >= 0.35 then o._ghostFrac = prevHP
                else o._ghostFrac = math.max(o._ghostFrac or prevHP, prevHP) end
                o._ghostT = ctx.now
            end
            if Config.ESPHealTick and prevHP and frac and frac > prevHP + 0.005 then o._healT = ctx.now end
            local style = Config.ESPBoxStyle or "Full Box"
            local corners = Config.ESPBoxBrackets or (style == "Corner Brackets")
            if Config.ESPBox and not corners then
                u.box.Visible = true
                paintBox(u, ctx)
                for i = 1, 8 do u.corners[i].Visible = false end
            elseif Config.ESPBox then
                u.box.Visible = false
                drawCorners(u, w, h, ctx)
            else
                u.box.Visible = false
                for i = 1, 8 do u.corners[i].Visible = false end
            end
            if Config.ESPBoxFill then
                u.fill.Visible = true
                u.fill.BackgroundTransparency = Config.ESPBoxFillTransparency or 0.75
                paint(u.fill, "BackgroundColor3", "fill", Config.ESPBoxColorMode or "Solid",
                      Config.ESPBoxFillColor, Config.ESPBoxGradA, Config.ESPBoxGradB,
                      Config.ESPGradientRotBox or 0, Config.ESPGradientSpeed, ctx.now)
            else u.fill.Visible = false end
            drawHealth(u, o, ctx, frac or 1, hp)
            drawText(u, o, player, ctx, dist, frac)
            drawFlags(u, o, player, ctx, char, frac)
            if Config.ESPLockChevron and o._primary then
                local pt = o._primT and math.clamp((ctx.now - o._primT) / 0.14, 0, 1) or 1
                local e = 1 - (1 - pt) * (1 - pt)
                u.chev.Visible = true
                styleLabel(u.chev, ctx.face, sizeFor(ctx.tsChip + 2, ctx.dmul), ctx.casing)
                u.chev.TextColor3 = Color3.fromRGB(255, 194, 75)
                u.chev.TextTransparency = 1 - e
                u.chev.Position = UDim2.new(0.5, 0, 0, -PAD - math.floor(sizeFor(ctx.tsName, ctx.dmul) * 1.7) - math.floor((1 - e) * 6))
            else u.chev.Visible = false end
            local pulseT, pulseCol = nil, nil
            if Config.ESPPeekAlert then
                local v = o._vis and true or false
                if v and not o._peekWasVis then o._peekT = ctx.now end
                o._peekWasVis = v
                if o._peekT and (ctx.now - o._peekT) < 0.14 then
                    pulseT = (ctx.now - o._peekT) / 0.14; pulseCol = Config.ESPBoxColor or WHITE
                end
            end
            if Config.ESPHealTick and o._healT and (ctx.now - o._healT) < 0.2 then
                pulseT = (ctx.now - o._healT) / 0.2; pulseCol = Color3.fromRGB(61, 224, 122)
            end
            if pulseT then
                local grow = math.floor(4 + pulseT * 6)
                u.pulse.Visible = true
                u.pulse.Size = UDim2.new(1, grow * 2, 1, grow * 2)
                u.pulseStroke.Color = pulseCol
                u.pulseStroke.Transparency = pulseT
            else u.pulse.Visible = false end
            if frac ~= nil then o._lastHP = frac end
        end
        drawSkeleton(u, o, ctx)
        if minX then drawAbsExtras(u, o, ctx, char, minX, minY, maxX, maxY)
        else
            u.tracer.Visible = false; u.dot.Visible = false; u.look.Visible = false
        end
        if Config.ESPThreatCount then
            local tsp = cam:WorldToViewportPoint(rootPos)
            local tOn = tsp.Z > 0 and tsp.X >= 0 and tsp.X <= vp.X and tsp.Y >= 0 and tsp.Y <= vp.Y
            if not tOn then ctx.threat = (ctx.threat or 0) + 1 end
        end
        if Config.ESPArrows then
            local ar = nd(o, "arrow", "Triangle")
            if ar then
                local asp = cam:WorldToViewportPoint(rootPos)
                local onScreen = asp.Z > 0 and asp.X >= 0 and asp.X <= vp.X and asp.Y >= 0 and asp.Y <= vp.Y
                if not onScreen then
                    local cen  = Vector2.new(vp.X * 0.5, vp.Y * 0.5)
                    local adir = Vector2.new(asp.X - cen.X, asp.Y - cen.Y)
                    if asp.Z <= 0 then adir = Vector2.new(-adir.X, -adir.Y) end
                    if adir.Magnitude < 1 then adir = Vector2.new(0, -1) end
                    adir = adir.Unit
                    local edge = cen + adir * (math.min(vp.X, vp.Y) * 0.38)
                    local perp = Vector2.new(-adir.Y, adir.X)
                    local aAlpha = 1 - (Config.ESPBoxTransparency or 0)
                    if Config.ESPArrowDistFade then
                        aAlpha = aAlpha * (1 - 0.65 * math.clamp(dist / math.max(Config.ESPMaxDistance, 1), 0, 1))
                    end
                    ar.Filled = false
                    ar.PointA = edge + adir * 20
                    ar.PointB = (edge - adir * 10) + perp * 8
                    ar.PointC = (edge - adir * 10) - perp * 8
                    ar.Color = Config.ESPBoxColor or WHITE
                    ar.Transparency = aAlpha; ar.Visible = true
                    if Config.ESPArrowDistLabel then
                        local dl = nd(o, "arrowDist", "Text")
                        if dl then
                            local lpos = edge - adir * 24
                            dl.Font = 3; dl.Size = 10; dl.Center = true; dl.Outline = true
                            dl.Color = Config.ESPInfoColor or WHITE
                            dl.Text = ("%dm"):format(dist)
                            dl.Position = Vector2.new(math.floor(lpos.X), math.floor(lpos.Y - 5))
                            dl.Transparency = aAlpha; dl.Visible = true
                        end
                    elseif o.arrowDist then o.arrowDist.Visible = false end
                else
                    ar.Visible = false
                    if o.arrowDist then o.arrowDist.Visible = false end
                end
            end
        elseif o.arrow then
            o.arrow.Visible = false
            if o.arrowDist then o.arrowDist.Visible = false end
        end
        if Config.ESPChams then
            local hl = ensureCham(o, char)
            if hl then
                local vc = Config.ESPChamsVisSplit
                    and (o._vis and (isTeammate(player) and Config.ColorTeam or Config.ColorEnemy)
                                 or (isTeammate(player) and Config.ColorTeamOcc or Config.ColorEnemyOcc))
                    or nil
                local state = vc or Config.ESPChamsFillColor
                local style2 = Config.ESPChamsStyle
                if style2 == "Shade" then
                    hl.FillColor = state; hl.OutlineColor = WHITE
                    hl.FillTransparency = 0.62; hl.OutlineTransparency = 0; hl.Enabled = true
                elseif style2 == "Neon" then
                    hl.FillColor = state; hl.OutlineColor = vc or Config.ESPChamsOutlineColor
                    hl.FillTransparency = 0.88; hl.OutlineTransparency = 0; hl.Enabled = true
                elseif style2 == "Ghost" then
                    hl.FillColor = state; hl.OutlineColor = WHITE
                    hl.FillTransparency = 1; hl.OutlineTransparency = 0.25; hl.Enabled = not o._vis
                else
                    hl.FillColor = vc or Config.ESPChamsFillColor
                    hl.OutlineColor = vc or Config.ESPChamsOutlineColor
                    hl.FillTransparency = Config.ESPChamsFillTransparency
                    hl.OutlineTransparency = Config.ESPChamsOutlineTransparency
                    hl.Enabled = true
                end
            end
        elseif o.cham then o.cham.Enabled = false end
    end
    local function _byDist(a, b)
        local ad, bd = a.d, b.d
        if ad ~= ad then return false end
        if bd ~= bd then return true end
        return ad < bd
    end
    local function doDeclutter(ctx)
        local myPos = ctx.myRoot and ctx.myRoot.Position
        local n = 0
        for player, o in pairs(State.ESPObjects) do
            o._declutter = false
            local c = _bboxCache[player]
            if c and o.root and o.root.Parent then
                n = n + 1
                local e = _dcArr[n]; if not e then e = {}; _dcArr[n] = e end
                e.o = o; e.c = c
                local rp = o.root.Position
                e.d = (myPos and rp) and (myPos - rp).Magnitude or math.huge
            end
        end
        for i = #_dcArr, n + 1, -1 do _dcArr[i] = nil end
        table.sort(_dcArr, _byDist)
        for i = 1, n - 1 do
            local A = _dcArr[i]; local ca = A.c
            local aMinX, aMinY, aMaxX, aMaxY = ca[1], ca[2], ca[3], ca[4]
            for j = i + 1, n do
                local B = _dcArr[j]
                if not B.o._declutter then
                    local cb = B.c
                    local ix = math.min(aMaxX, cb[3]) - math.max(aMinX, cb[1])
                    local iy = math.min(aMaxY, cb[4]) - math.max(aMinY, cb[2])
                    if ix > 0 and iy > 0 then
                        local barea = (cb[3] - cb[1]) * (cb[4] - cb[2])
                        if barea > 0 and (ix * iy) / barea > 0.6 then B.o._declutter = true end
                    end
                end
            end
        end
    end
    local function renderRadar(ctx)
        if not Config.ESPRadar then
            for _, d in ipairs(_radarO.drawings) do d.Visible = false end
            return
        end
        if (_espFrame % 2) ~= 0 then return end
        local vp = ctx.vp; if not vp then return end
        local R     = math.max((Config.ESPRadarSize or 180) * 0.5, 10)
        local inset = Config.ESPRadarInset or 20
        local cx    = vp.X - inset - R
        local cy    = inset + R
        local disc = nd(_radarO, "disc", "Circle")
        if disc then
            disc.Filled = true; disc.NumSides = 32; disc.Radius = R
            disc.Position = Vector2.new(cx, cy)
            disc.Color = Color3.fromRGB(13, 18, 25)
            disc.Transparency = 0.62; disc.Visible = true
        end
        local rring = nd(_radarO, "rangeRing", "Circle")
        if rring then
            rring.Filled = false; rring.Thickness = 1; rring.NumSides = 28; rring.Radius = R * 0.5
            rring.Position = Vector2.new(cx, cy); rring.Color = WHITE
            rring.Transparency = 0.10; rring.Visible = true
        end
        local myPos = ctx.myRoot and ctx.myRoot.Position
        if not myPos then return end
        local rot = 0
        if Config.ESPRadarRotate then
            local look = cam.CFrame.LookVector
            rot = -math.atan2(look.X, look.Z)
        end
        local cosR, sinR = math.cos(rot), math.sin(rot)
        local range = math.max(Config.ESPRadarRange or 250, 1)
        local ticks = ndArr(_radarO, "ticks", "Line", 4)
        for i = 0, 3 do
            local L = ticks[i + 1]
            if L then
                local ang = i * (math.pi * 0.5) + rot
                local dxT, dyT = math.sin(ang), -math.cos(ang)
                L.From = Vector2.new(cx + dxT * (R - 6), cy + dyT * (R - 6))
                L.To   = Vector2.new(cx + dxT * R, cy + dyT * R)
                L.Thickness = 1
                if i == 0 then L.Color = Color3.fromRGB(255, 194, 75); L.Transparency = 0.9
                else L.Color = WHITE; L.Transparency = 0.35 end
                L.Visible = true
            end
        end
        if Config.ESPRadarGrid then
            local grid = ndArr(_radarO, "grid", "Line", 2)
            for gi = 1, 2 do
                local L = grid[gi]
                if L then
                    local ang = rot + (gi - 1) * (math.pi * 0.5)
                    local gx, gy = math.sin(ang), -math.cos(ang)
                    L.From = Vector2.new(cx - gx * R, cy - gy * R)
                    L.To   = Vector2.new(cx + gx * R, cy + gy * R)
                    L.Thickness = 1
                    L.Color = Color3.fromRGB(168, 180, 192)
                    L.Transparency = 0.10
                    L.Visible = true
                end
            end
        elseif _radarO.grid then
            for _, L in ipairs(_radarO.grid) do L.Visible = false end
        end
        if Config.ESPRadarSweep then
            local sw = ndArr(_radarO, "sweep", "Line", 3)
            local baseAng = ((ctx.now or 0) % 4) / 4 * (math.pi * 2) + rot
            for si = 1, 3 do
                local L = sw[si]
                if L then
                    local ang = baseAng - (si - 1) * 0.12
                    local sx, sy = math.sin(ang), -math.cos(ang)
                    L.From = Vector2.new(cx, cy)
                    L.To   = Vector2.new(cx + sx * (R - 2), cy + sy * (R - 2))
                    L.Thickness = si == 1 and 2 or 1
                    L.Color = Color3.fromRGB(255, 194, 75)
                    L.Transparency = si == 1 and 0.35 or (si == 2 and 0.18 or 0.08)
                    L.Visible = true
                end
            end
        elseif _radarO.sweep then
            for _, L in ipairs(_radarO.sweep) do L.Visible = false end
        end
        local idx = 0
        local closestIdx, closestH = nil, math.huge
        local closestPx, closestPy, closestDotR = 0, 0, 0
        for player, o in pairs(State.ESPObjects) do
            local root = o.root
            if root and root.Parent then
                local rp = root.Position
                local dx = rp.X - myPos.X
                local dz = rp.Z - myPos.Z
                local sx = (dx / range) * R
                local sz = (dz / range) * R
                local rx = sx * cosR - sz * sinR
                local rz = sx * sinR + sz * cosR
                local mag = math.sqrt(rx * rx + rz * rz)
                if mag > R then rx = rx / mag * R; rz = rz / mag * R end
                idx = idx + 1
                local px, py = cx + rx, cy + rz
                local hdist  = math.sqrt(dx * dx + dz * dz)
                local dotR   = math.clamp(5 - (hdist / range) * 2, 3, 5)
                local team   = isTeammate(player)
                local filled = (not Config.ESPRadarVisSplit) or (o._vis and true or false)
                local col
                if Config.ESPRadarVisSplit and not o._vis then
                    col = team and Config.ColorTeamOcc or Config.ColorEnemyOcc
                else
                    col = team and Config.ColorTeam or Config.ColorEnemy
                end
                if hdist < closestH then closestH = hdist; closestIdx = idx; closestPx = px; closestPy = py; closestDotR = dotR end
                local dsh = ndArr(_radarO, "dotShadow", "Circle", idx)
                local sD = dsh[idx]
                if sD then
                    sD.Filled = true; sD.NumSides = 12; sD.Radius = dotR + 1
                    sD.Position = Vector2.new(px, py); sD.Color = BLACK
                    sD.Transparency = 0.6; sD.Visible = true
                end
                local dots = ndArr(_radarO, "dots", "Circle", idx)
                local d = dots[idx]
                if d then
                    d.Filled = filled
                    if not filled then d.Thickness = 1 end
                    d.NumSides = 12; d.Radius = dotR
                    d.Position = Vector2.new(px, py); d.Color = col
                    d.Transparency = 1; d.Visible = true
                end
                local chevs = ndArr(_radarO, "chev", "Triangle", idx)
                local cv = chevs[idx]
                if cv then
                    local dy = rp.Y - myPos.Y
                    if dy > 6 or dy < -6 then
                        local dir = dy > 6 and -1 or 1
                        cv.Filled = false
                        cv.PointA = Vector2.new(px, py + dir * (dotR + 4))
                        cv.PointB = Vector2.new(px - 3, py + dir * (dotR + 1))
                        cv.PointC = Vector2.new(px + 3, py + dir * (dotR + 1))
                        cv.Color = col; cv.Transparency = 1; cv.Visible = true
                    else cv.Visible = false end
                end
            end
        end
        local nearRing = nd(_radarO, "nearRing", "Circle")
        if nearRing then
            if closestIdx then
                local pulse = 1 + 0.28 * (0.5 + 0.5 * math.sin((ctx.now or 0) * 1.6 * math.pi * 2))
                nearRing.Filled = false; nearRing.Thickness = 1.2; nearRing.NumSides = 24
                nearRing.Radius = (closestDotR + 3) * pulse
                nearRing.Position = Vector2.new(closestPx, closestPy)
                nearRing.Color = Color3.fromRGB(255, 194, 75)
                nearRing.Transparency = 1; nearRing.Visible = true
            else
                nearRing.Visible = false
            end
        end
        if _radarO.dots      then for i = idx + 1, #_radarO.dots do _radarO.dots[i].Visible = false end end
        if _radarO.dotShadow then for i = idx + 1, #_radarO.dotShadow do _radarO.dotShadow[i].Visible = false end end
        if _radarO.chev      then for i = idx + 1, #_radarO.chev do _radarO.chev[i].Visible = false end end
        local selfTri = nd(_radarO, "self", "Triangle")
        if selfTri then
            selfTri.Filled = true
            selfTri.PointA = Vector2.new(cx, cy - 5)
            selfTri.PointB = Vector2.new(cx - 4, cy + 4)
            selfTri.PointC = Vector2.new(cx + 4, cy + 4)
            selfTri.Color = Color3.fromRGB(255, 194, 75); selfTri.Transparency = 1; selfTri.Visible = true
        end
        local rimShadow = nd(_radarO, "rimShadow", "Circle")
        if rimShadow then
            rimShadow.Filled = false; rimShadow.Thickness = 2; rimShadow.NumSides = 32; rimShadow.Radius = R
            rimShadow.Position = Vector2.new(cx, cy); rimShadow.Color = BLACK
            rimShadow.Transparency = 0.55; rimShadow.Visible = true
        end
        local rim = nd(_radarO, "rim", "Circle")
        if rim then
            rim.Filled = false; rim.Thickness = 1; rim.NumSides = 32; rim.Radius = R
            rim.Position = Vector2.new(cx, cy)
            rim.Color = Color3.fromRGB(168, 180, 192)
            rim.Transparency = 0.45; rim.Visible = true
        end
    end
    local function render()
        local _now = tick()
        local _dt  = _now - _lastRenderT
        -- OPT FPS: perf ~10 Hz ESP, normal ~30 Hz (menos carga en RenderStepped)
        local espHz = 0.033
        if config and config.ESPPerfMode ~= false then
            espHz = tonumber(config.ESPUpdateInterval) or 0.1 -- ~10 fps ESP
        end
        if _now - _lastRenderT < espHz then return end
        _lastRenderT = _now
        _espFrame = _espFrame + 1
        _createBudget = CREATE_BUDGET_PER_FRAME
        if not Config.ESP then
            State.PrimaryTarget = nil
            for _, o in pairs(State.ESPObjects) do hideAll(o, true) end
            for _, d in ipairs(_radarO.drawings) do d.Visible = false end
            for _, d in ipairs(_module.drawings) do d.Visible = false end
            return
        end
        local ctx = _ctx
        ctx.vp     = cam.ViewportSize
        ctx.myRoot = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
        ctx.dt     = math.clamp(_dt, 0, 0.1)
        ctx.now    = _now
        ctx.threat = 0
        ctx.face   = faceFor(Config.ESPFont or "Code")
        local ts = (ctx.vp.Y / BASE_H) * (Config.ESPTextScale or 1)
        ctx.tsName = typePx(Config.ESPTextSize or 14, ts)
        ctx.tsInfo = typePx(Config.ESPInfoTextSize or 12, ts)
        ctx.tsHp   = typePx(Config.ESPHealthTextSize or 11, ts)
        ctx.tsChip = typePx(10, ts)
        ctx.casing = math.floor(math.clamp(Config.ESPTextCasing or 1, 0, 4))
        -- declutter solo fuera de perf (caro)
        if config and config.ESPPerfMode == false and Config.ESPDeclutter and (ctx.now - _dcLastT) >= 0.2 then
            _dcLastT = ctx.now; pcall(doDeclutter, ctx)
        end
        local cap = Config.ESPMaxPlayers or 0
        if config and config.ESPPerfMode ~= false then
            cap = math.min(cap > 0 and cap or 8, 8)
        end
        if cap and cap > 0 then
            local arr = _renderArr
            local n = 0
            local myPos = ctx.myRoot and ctx.myRoot.Position
            for player, o in pairs(State.ESPObjects) do
                n = n + 1
                local e = arr[n]; if not e then e = {}; arr[n] = e end
                e.p = player; e.o = o
                local rp = o.root and o.root.Parent and o.root.Position
                e.d = (myPos and rp) and (myPos - rp).Magnitude or math.huge
            end
            for i = #arr, n + 1, -1 do arr[i] = nil end
            pcall(table.sort, arr, _byDist)
            if Config.ESPPrimaryEmphasis or Config.ESPLockChevron or Config.FXTargetInfo then
                for i = 1, n do arr[i].o._primary = false end
                local bestO = arr[1] and arr[1].o
                local bestD = (arr[1] and arr[1].d) or math.huge
                local lockO, lockD
                for i = 1, n do if arr[i].o == _lock.obj then lockO = arr[i].o; lockD = arr[i].d; break end end
                local locked = resolveLock(bestO, bestD, lockO, lockD, ctx.now)
                State.PrimaryTarget = nil
                if locked then
                    locked._primary = true
                    for i = 1, n do if arr[i].o == locked then State.PrimaryTarget = arr[i].p break end end
                end
            else
                State.PrimaryTarget = nil
            end
            for i = 1, n do
                local e = arr[i]
                if i <= cap then
                    e.o._dist = e.d
                    pcall(renderPlayer, e.p, e.o)
                else
                    hideAll(e.o)
                end
            end
        else
            if Config.ESPPrimaryEmphasis or Config.ESPLockChevron or Config.FXTargetInfo then
                local myPos = ctx.myRoot and ctx.myRoot.Position
                local bestD, bestO = math.huge, nil
                local lockO, lockD
                for player, o in pairs(State.ESPObjects) do
                    o._primary = false
                    local rp = o.root and o.root.Parent and o.root.Position
                    local d = (myPos and rp) and (myPos - rp).Magnitude or math.huge
                    if d < bestD then bestD = d; bestO = o end
                    if o == _lock.obj then lockO = o; lockD = d end
                end
                local locked = resolveLock(bestO, bestD, lockO, lockD, ctx.now)
                State.PrimaryTarget = nil
                if locked then
                    locked._primary = true
                    for player, o in pairs(State.ESPObjects) do
                        if o == locked then State.PrimaryTarget = player break end
                    end
                end
            else
                State.PrimaryTarget = nil
            end
            local perf = config and config.ESPPerfMode ~= false
            local idx = 0
            for player, o in pairs(State.ESPObjects) do
                o._dist = nil
                idx = idx + 1
                -- en perf: half players each frame
                if not (perf and ((idx + _espFrame) % 2 == 1)) then
                    pcall(renderPlayer, player, o)
                end
            end
        end
        if not (config and config.ESPPerfMode ~= false) then
            pcall(renderRadar, ctx)
        end
        if Config.ESPThreatCount then
            pcall(function()
                local t = nd(_module, "threat", "Text")
                if t then
                    local cnt = ctx.threat or 0
                    if cnt > 0 then
                        t.Font = 3; t.Size = 14; t.Center = true; t.Outline = true
                        t.Color = Config.ColorEnemy
                        t.Text = cnt .. " <"
                        t.Position = Vector2.new(math.floor(ctx.vp.X * 0.5), math.floor(ctx.vp.Y * 0.5 + 28))
                        t.Transparency = 1; t.Visible = true
                    else t.Visible = false end
                end
            end)
        elseif _module.threat then _module.threat.Visible = false end
    end
    function ESP.init()
        for _, p in ipairs(getSafePlayers()) do
            if p ~= lp then
                p.CharacterAdded:Connect(function()
                    task.wait(0.5); if Config.ESP then buildESP(p) end
                end)
            end
        end
        Players.PlayerAdded:Connect(function(p)
            p.CharacterAdded:Connect(function()
                task.wait(0.5); if Config.ESP then buildESP(p) end
            end)
        end)
        Players.PlayerRemoving:Connect(function(p) cleanESP(p) end)
        if Config.ESP and not _renderConn then
            _renderConn = RunService.Heartbeat:Connect(render)
            ESP.rebuildAll()
        end
    end
    function ESP.enable()
        Config.ESP = true
        -- hooks de respawn (si solo enable y no init, no se reconstruía al morir)
        if not ESP._hooksReady then
            ESP._hooksReady = true
            for _, p in ipairs(getSafePlayers()) do
                if p ~= lp then
                    p.CharacterAdded:Connect(function()
                        task.spawn(function()
                            for _, d in ipairs({0.3, 0.8, 1.5, 2.5}) do
                                task.wait(d)
                                if not Config.ESP then return end
                                if p.Character then
                                    pcall(function() buildESP(p) end)
                                end
                            end
                        end)
                    end)
                end
            end
            Players.PlayerAdded:Connect(function(p)
                p.CharacterAdded:Connect(function()
                    task.spawn(function()
                        for _, d in ipairs({0.3, 0.8, 1.5, 2.5}) do
                            task.wait(d)
                            if not Config.ESP then return end
                            if p.Character then pcall(function() buildESP(p) end) end
                        end
                    end)
                end)
            end)
            Players.PlayerRemoving:Connect(function(p)
                pcall(function() cleanESP(p) end)
            end)
        end
        ESP.rebuildAll()
        if not _renderConn then _renderConn = RunService.Heartbeat:Connect(render) end
        -- forzar rebuild periódico por si se pierde un respawn
        if not ESP._refreshConn then
            ESP._refreshConn = true
            task.spawn(function()
                while Config.ESP do
                    task.wait(2)
                    if not Config.ESP then break end
                    for _, p in ipairs(getSafePlayers()) do
                        if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                            if not State.ESPObjects[p] or not State.ESPObjects[p].root or not State.ESPObjects[p].root.Parent then
                                pcall(function() buildESP(p) end)
                            end
                        end
                    end
                end
                ESP._refreshConn = false
            end)
        end
    end
    function ESP.disable()
        Config.ESP = false; ESP.rebuildAll()
        State.PrimaryTarget = nil
        if _renderConn then _renderConn:Disconnect(); _renderConn = nil end
    end
    function ESP.unload()
        if _renderConn then _renderConn:Disconnect(); _renderConn = nil end
        State.PrimaryTarget = nil
        for p in pairs(State.ESPObjects) do cleanESP(p) end
        for _, pool in ipairs({ _radarO, _module }) do
            for _, d in ipairs(pool.drawings) do
                pcall(function() if d.Remove then d:Remove() end end)
            end
            pool.drawings = {}
            pool.disc, pool.rim, pool.self, pool.dots, pool.chev, pool.threat = nil, nil, nil, nil, nil, nil
            pool.rangeRing, pool.ticks, pool.dotShadow, pool.rimShadow, pool.nearRing = nil, nil, nil, nil, nil
            pool.grid, pool.sweep = nil, nil
        end
        if _gui then pcall(function() _gui:Destroy() end); _gui = nil; _absLayer = nil end
    end
end)()


-- Wire Start/Stop al módulo LuaHook
StartESP = function()
    config.ESPEnabled = true
    config.ESP = true
    -- OPT: modo rendimiento fuerza opciones ligeras
    if config.ESPPerfMode ~= false then
        config.ESPSkeleton = false
        config.ESPFlags = false
        config.ESPRadar = false
        config.ESPChams = false
        config.ESPTracers = false
        config.ESPGlow = false
        config.ESPDeclutter = false
        config.ESPFadeIn = false
        config.ESPMaxPlayers = math.min(tonumber(config.ESPMaxPlayers) or 8, 8)
        config.ESPUpdateInterval = tonumber(config.ESPUpdateInterval) or 0.1
        pcall(function()
            Config.ESPSkeleton = false
            Config.ESPFlags = false
            Config.ESPRadar = false
            Config.ESPChams = false
            Config.ESPTracers = false
            Config.ESPDeclutter = false
            Config.ESPFadeIn = false
            Config.ESPMaxPlayers = 8
        end)
    end
    -- sync flags UI → Config/LuaHook
    config.ESPBox = config.ESPShowBox ~= false
    config.ESPName = config.ESPShowName ~= false
    config.ESPDistance = config.ESPShowDist ~= false
    config.ESPHealth = config.ESPShowHP ~= false
    config.ESPHealthBar = config.ESPShowHP ~= false
    config.ESPSkeleton = config.ESPSkeleton == true
    config.ESPTracers = config.ESPTracers == true
    -- Team check OFF por defecto al activar si no hay teams reales (evita ESP vacío)
    if config.ESPTeamCheck == nil then config.ESPTeamCheck = false end
    -- Distancia infinita
    local md = tonumber(config.ESPMaxDistance)
    if not md or md <= 0 then config.ESPMaxDistance = 100000 end
    if config.ESPBoxStyle == "Corner" or config.ESPBoxStyle == "Corner Brackets" then
        config.ESPBoxStyle = "Corner Brackets"
        config.ESPBoxBrackets = true
    elseif config.ESPBoxStyle == "Full" or config.ESPBoxStyle == "Full Box" then
        config.ESPBoxStyle = "Full Box"
        config.ESPBoxBrackets = false
    end
    pcall(function()
        if ESP and ESP.enable then
            ESP.enable()
        elseif ESP and ESP.init then
            ESP.init()
        end
    end)
end

StopESP = function()
    config.ESPEnabled = false
    config.ESP = false
    pcall(function()
        if ESP and ESP.disable then
            ESP.disable()
        elseif ESP and ESP.unload then
            ESP.unload()
        end
    end)
    pcall(function() SafeDisconnect("ESP") end)
end

function pushBulletTracer(fromPos, toPos)
    if not config.BulletTracerEnabled then return end
    if not fromPos or not toPos then return end
    BulletTracers = BulletTracers or {}
    while #BulletTracers >= 16 do
        local old = table.remove(BulletTracers, 1)
        if old then
            pcall(function() if old.beam then old.beam:Destroy() end end)
            pcall(function() if old.a0 then old.a0:Destroy() end end)
            pcall(function() if old.a1 then old.a1:Destroy() end end)
            pcall(function() if old.light then old.light:Destroy() end end)
            pcall(function() if old.line and old.line.Remove then old.line:Remove() end end)
        end
    end
    local col = (fovColor and fovColor(config.BulletTracerColor or "Cyan")) or Color3.fromRGB(80,220,255)
    local style = tostring(config.BulletTracerStyle or "Line")
    local size = tonumber(config.BulletTracerThickness) or 1.5
    local glow = tonumber(config.BulletTracerGlow) or 0.45
    local dur = tonumber(config.BulletTracerDuration) or 2.5
    local fade = tonumber(config.BulletTracerFade) or 0.5
    local a0 = Instance.new("Attachment"); a0.WorldPosition = fromPos; a0.Parent = workspace.Terrain
    local a1 = Instance.new("Attachment"); a1.WorldPosition = toPos; a1.Parent = workspace.Terrain
    local beam = Instance.new("Beam")
    beam.Attachment0, beam.Attachment1 = a0, a1
    beam.Color = ColorSequence.new(col)
    local bw = (style == "Laser" and 0.02) or (style == "Glow" and 0.18) or 0.05
    beam.Width0 = bw * size
    beam.Width1 = bw * size * (style == "Glow" and 1.35 or 0.65)
    beam.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.75, 0.12),
        NumberSequenceKeypoint.new(1, 0.5),
    })
    beam.FaceCamera = true
    beam.LightEmission = glow
    beam.LightInfluence = math.clamp(1 - glow, 0, 1)
    beam.Parent = workspace.Terrain
    local light
    if glow > 0.25 then
        light = Instance.new("PointLight")
        light.Brightness = glow * 2
        light.Range = 8 * size
        light.Color = col
        light.Parent = a1
    end
    table.insert(BulletTracers, {
        beam = beam, a0 = a0, a1 = a1, light = light,
        created = tick(), die = tick() + dur, fade = fade, from = fromPos, to = toPos,
    })
end

function updateBulletTracers()
    if not BulletTracers then return end
    local now = tick()
    for i = #BulletTracers, 1, -1 do
        local t = BulletTracers[i]
        if now >= t.die then
            pcall(function() if t.beam then t.beam:Destroy() end end)
            pcall(function() if t.a0 then t.a0:Destroy() end end)
            pcall(function() if t.a1 then t.a1:Destroy() end end)
            pcall(function() if t.light then t.light:Destroy() end end)
            table.remove(BulletTracers, i)
        else
            local life = t.die - now
            local fadeT = t.fade or 0.5
            if life < fadeT and t.beam then
                local fp = 1 - (life / fadeT)
                pcall(function()
                    t.beam.Transparency = NumberSequence.new({
                        NumberSequenceKeypoint.new(0, fp),
                        NumberSequenceKeypoint.new(0.8, math.min(1, fp + 0.15)),
                        NumberSequenceKeypoint.new(1, 1),
                    })
                end)
            end
            if t.a0 and t.from then pcall(function() t.a0.WorldPosition = t.from end) end
            if t.a1 and t.to then pcall(function() t.a1.WorldPosition = t.to end) end
        end
    end
end




-- ==================== WEATHER + EMOTES ====================

-- ==================== WEATHER (LuaHook) + Config bridge ====================
do
    local _cfg = config
    -- ensure keys
    _cfg.WeatherEnabled = _cfg.WeatherEnabled or false
    _cfg.Weather = false
    _cfg.WeatherType = _cfg.WeatherType or "Rain"
    _cfg.WeatherIntensity = _cfg.WeatherIntensity or 1
    _cfg.WeatherSoundVolume = _cfg.WeatherSoundVolume or 0.6
    _cfg.WeatherStorm = _cfg.WeatherStorm or false
    _cfg.WeatherStormMin = _cfg.WeatherStormMin or 8
    _cfg.WeatherStormVar = _cfg.WeatherStormVar or 6
    _cfg.WeatherMeteors = _cfg.WeatherMeteors or false
    _cfg.WeatherMeteorRate = _cfg.WeatherMeteorRate or 1
    _cfg.WeatherShootingStars = _cfg.WeatherShootingStars or false
    _cfg.WeatherStarRate = _cfg.WeatherStarRate or 1
    _cfg.WeatherPuddles = _cfg.WeatherPuddles or false
    _cfg.WeatherMood = _cfg.WeatherMood ~= false
    _cfg.WeatherClockDial = _cfg.WeatherClockDial or false
    _cfg.WeatherRainbow = _cfg.WeatherRainbow or false
    _cfg.WeatherGodRays = _cfg.WeatherGodRays or false
    _cfg.SkyboxPreset = _cfg.SkyboxPreset or "Off"
    _cfg.ThirdPersonEnabled = _cfg.ThirdPerson or false
    _cfg.ThirdPersonDistance = _cfg.ThirdPersonDistance or 12
end

-- LuaHook Weather expects global-ish Config + lp
local Config = setmetatable({}, {
    __index = function(_, k)
        if k == "Weather" then return config.WeatherEnabled or config.Weather end
        if k == "ThirdPersonEnabled" then return config.ThirdPerson or config.ThirdPersonEnabled end
        return config[k]
    end,
    __newindex = function(_, k, v)
        if k == "Weather" then
            config.WeatherEnabled = v and true or false
            config.Weather = v and true or false
        elseif k == "ThirdPersonEnabled" then
            config.ThirdPerson = v and true or false
            config.ThirdPersonEnabled = v and true or false
        else
            config[k] = v
        end
    end,
})
local lp = LocalPlayer
local Workspace = workspace
if not Camera then Camera = workspace.CurrentCamera end

local Weather = {}
;(function()
    local SoundService = game:GetService("SoundService")
    local TweenService = game:GetService("TweenService")
    local function cfg(key, default)
        local v = Config[key]
        if v == nil then return default end
        return v
    end
    local function iAmt()
        return math.clamp(cfg("WeatherIntensity", 1), 0.15, 2)
    end
    local function iRate(I, kLo, kHi)
        if I < 1 then
            return math.exp(kLo * (I - 1))
        end
        return math.exp(kHi * (I - 1))
    end
    local function evRate(key)
        return math.clamp(cfg(key, 1), 0.25, 3)
    end
    local IR = {
        Snow      = { lo = 1.75, hi = 0.85, acc = 0.45, sz = 0.08, alp = 0.35, gst = 0.30 },
        Petals    = { lo = 1.75, hi = 0.85, acc = 0.40, sz = 0.08, alp = 0.30, gst = 0.30 },
        Autumn    = { lo = 1.75, hi = 0.85, acc = 0.40, sz = 0.08, alp = 0.30, gst = 0.30 },
        Mist      = { lo = 1.35, hi = 1.10, acc = 0.30, sz = 0.25, alp = 0.75, gst = 0.20 },
        Ash       = { lo = 1.45, hi = 1.05, acc = 0.35, sz = 0.15, alp = 0.55, gst = 0.20 },
        Sandstorm = { lo = 1.35, hi = 1.60, acc = 0.45, sz = 0.30, alp = 0.75, gst = 0.20 },
        Embers    = { lo = 1.45, hi = 1.05, acc = 0.30, sz = 0.10, alp = 0.25, gst = 0.20 },
        Fireflies = { lo = 1.45, hi = 1.00, acc = 0.10, sz = 0.08, alp = 0.20, gst = 0.15 },
    }
    local IR_DEF = { lo = 1.6, hi = 0.9, acc = 0.3, sz = 0.10, alp = 0.25, gst = 0.25 }
    local SET_LO, SET_HI = 0.75, 0.45
    local Vec   = Vector3.new
    local WHITE = Color3.new(1, 1, 1)
    local TX_SOFT  = "rbxasset://textures/particles/smoke_main.dds"
    local TX_SPARK = "rbxasset://textures/particles/sparkles_main.dds"
    local TX_FLAKE    = "rbxassetid://101749163393113"
    local TX_SNOWSOFT = "rbxassetid://78582616787441"
    local TX_PUFF     = "rbxassetid://77935321198144"
    local TX_PETAL    = "rbxassetid://243344623"
    local TX_LEAFA    = "rbxassetid://8590047664"
    local TX_LEAFB    = "rbxassetid://5970677338"
    local TX_LEAFC    = "rbxassetid://9239040931"
    local TX_DUST     = "rbxassetid://341828512"
    local _curType = nil
    local Wind = { x = 0, z = 0 }
    ;(function()
        local baseAng = math.random() * 6.283
        local gustT0, gustDur, gustAmp = 0, 1, 0
        local nextGustT = math.huge
        local cbs = {}
        function Wind.onGust(fn)
            cbs[#cbs + 1] = fn
        end
        function Wind.reset(now)
            nextGustT = now + 8 + math.random() * 17
            gustT0, gustAmp = 0, 0
            Wind.x, Wind.z = 0, 0
        end
        function Wind.update(now)
            if now >= nextGustT then
                gustT0 = now
                gustDur = 2 + math.random() * 2
                gustAmp = 1 + math.random()
                nextGustT = now + 8 + math.random() * 17
                for i = 1, #cbs do
                    pcall(cbs[i], gustDur, 1 + gustAmp)
                end
            end
            local ang = baseAng + 0.6 * math.sin(now * 0.013) + 0.9 * math.sin(now * 0.031 + 2.6)
            local str = 1 + 0.35 * math.sin(now * 0.17) + 0.22 * math.sin(now * 0.41 + 1.3)
                          + 0.15 * math.sin(now * 0.07 + 4.1)
            local ga = (now - gustT0) / gustDur
            if gustAmp > 0 and ga < 1 then
                str = str * (1 + gustAmp * math.sin(ga * math.pi))
            end
            Wind.x = math.cos(ang) * str
            Wind.z = math.sin(ang) * str
        end
        function Wind.vec()
            return Vec(Wind.x, 0, Wind.z)
        end
    end)()
    local _folder = nil
    local function getFolder()
        if _folder and _folder.Parent then return _folder end
        local f = Instance.new("Folder"); f.Name = "_wx"; f:SetAttribute("WX_Custom", true)
        f.Parent = Workspace
        _folder = f; return f
    end
    local _rain = { drops = {}, conn = nil, folder = nil, dir = nil, applied = nil }
    local RAIN_DIR      = Vec(-0.16, -1, 0.05).Unit
    local RAIN_LEAN_K   = 0.12
    local RAIN_LEAN_MAX = 0.2126
    local RAIN_RADIUS = 70
    local RAIN_TOP    = 70
    local RAIN_BOT    = -28
    local RAIN_MAX    = 420
    local function rainFolder()
        if _rain.folder and _rain.folder.Parent then return _rain.folder end
        local f = Instance.new("Folder"); f.Name = "_wxRain"; f:SetAttribute("WX_Custom", true)
        f.Parent = getFolder()
        _rain.folder = f; return f
    end
    local function makeDrop(parent, streakLen, width, color, glow, transp)
        local part = Instance.new("Part")
        part.Anchored = true; part.CanCollide = false; part.CanQuery = false; part.CanTouch = false
        part.CastShadow = false; part.Massless = true; part.Transparency = 1; part.Size = Vec(0.05,0.05,0.05)
        part:SetAttribute("WX_Custom", true)
        local a0 = Instance.new("Attachment"); a0.Parent = part
        local a1 = Instance.new("Attachment"); a1.Position = _rain.dir * streakLen; a1.Parent = part
        local beam = Instance.new("Beam")
        beam.Attachment0 = a0; beam.Attachment1 = a1
        beam.Segments = 1; beam.FaceCamera = true
        beam.Width0 = width; beam.Width1 = width * 0.55
        local emission = 0.35
        if glow then
            emission = 1
        end
        beam.LightEmission = emission
        beam.LightInfluence = 0
        beam.Color = ColorSequence.new(color)
        beam.Transparency = NumberSequence.new(transp)
        beam.Parent = part
        part.Parent = parent
        return part, a1, beam
    end
    local function seedDrop(d, camPos)
        local ang = math.random() * math.pi * 2
        local rad = math.sqrt(math.random()) * RAIN_RADIUS
        local y   = camPos.Y + RAIN_TOP - math.random() * (RAIN_TOP - RAIN_BOT)
        d.pos = Vec(camPos.X + math.cos(ang) * rad, y, camPos.Z + math.sin(ang) * rad)
    end
    local function newDrop(folder, i, camPos)
        local base, width, color, transp, spd0
        if (i % 3) ~= 0 then
            base, width, transp = 5 + math.random() * 3, 0.10, 0.22
            spd0 = 150 + math.random() * 30
            color = Color3.fromRGB(180, 202, 232)
        else
            base, width, transp = 3 + math.random() * 2, 0.06, 0.55
            spd0 = 120 + math.random() * 30
            color = Color3.fromRGB(158, 182, 214)
        end
        local part, a1, beam = makeDrop(folder, base, width, color, false, transp)
        local d = { part = part, a1 = a1, beam = beam, base = base, len = base,
                    w0 = width, t0 = transp, spd0 = spd0, spd = spd0 }
        seedDrop(d, camPos)
        part.CFrame = CFrame.new(d.pos)
        return d
    end
    local function tuneRain()
        local folder = rainFolder()
        local I = iAmt()
        local n = math.clamp(math.floor(150 * iRate(I, 1.75, 0.85)), 16, RAIN_MAX)
        local drops = _rain.drops
        local camPos = Vec(0, 0, 0)
        if Camera then
            camPos = Camera.CFrame.Position
        end
        for i = #drops, n + 1, -1 do
            drops[i].part:Destroy()
            drops[i] = nil
        end
        for i = #drops + 1, n do
            drops[i] = newDrop(folder, i, camPos)
        end
        local sm = 0.55 + 0.45 * I
        local wm = 0.85 + 0.15 * I
        local am = 0.78 + 0.22 * I
        local dir = _rain.applied or _rain.dir or RAIN_DIR
        for i = 1, n do
            local d = drops[i]
            d.spd = d.spd0 * sm
            d.len = d.base * sm
            d.a1.Position = dir * d.len
            local w = d.w0 * wm
            d.beam.Width0 = w
            d.beam.Width1 = w * 0.55
            d.beam.Transparency = NumberSequence.new(math.clamp(1 - (1 - d.t0) * am, 0.02, 1))
        end
    end
    local function buildRain()
        if _rain.folder then pcall(function() _rain.folder:Destroy() end); _rain.folder = nil end
        table.clear(_rain.drops)
        if not _rain.dir then _rain.dir = RAIN_DIR end
        _rain.applied = _rain.dir
        tuneRain()
    end
    local function startRain()
        if _rain.conn then return end
        _rain.conn = RunService.Heartbeat:Connect(function(dt)
            if not Config.Weather or _curType ~= "Rain" then return end
            local cam = Camera; if not cam then return end
            local camPos = cam.CFrame.Position
            local lx, lz = Wind.x * RAIN_LEAN_K, Wind.z * RAIN_LEAN_K
            local lm = math.sqrt(lx * lx + lz * lz)
            if lm > RAIN_LEAN_MAX then
                local s = RAIN_LEAN_MAX / lm
                lx, lz = lx * s, lz * s
            end
            local step = _rain.dir:Lerp(Vec(lx, -1, lz).Unit, math.min(dt * 2, 1)).Unit
            _rain.dir = step
            if (step - _rain.applied).Magnitude > 0.015 then
                _rain.applied = step
                for i = 1, #_rain.drops do
                    local d = _rain.drops[i]
                    d.a1.Position = step * d.len
                end
            end
            local r2 = RAIN_RADIUS * RAIN_RADIUS
            for i = 1, #_rain.drops do
                local d = _rain.drops[i]
                local p = d.pos + step * (d.spd * dt)
                local relX, relY, relZ = p.X - camPos.X, p.Y - camPos.Y, p.Z - camPos.Z
                if relY < RAIN_BOT or (relX * relX + relZ * relZ) > r2 then
                    seedDrop(d, camPos)
                    p = d.pos
                end
                d.pos = p
                if d.part then d.part.CFrame = CFrame.new(p) end
            end
        end)
    end
    local function stopRain()
        if _rain.conn then _rain.conn:Disconnect(); _rain.conn = nil end
        if _rain.folder then pcall(function() _rain.folder:Destroy() end); _rain.folder = nil end
        table.clear(_rain.drops)
        _rain.dir = nil; _rain.applied = nil
    end
    local ND = Enum.NormalId
    local function flutterSeq(lo, hi, n, env, flip)
        local kps = table.create(n + 1)
        for i = 0, n do
            local v = lo
            if (i + flip) % 2 == 1 then
                v = hi
            end
            kps[i + 1] = NumberSequenceKeypoint.new(i / n, v, env)
        end
        return NumberSequence.new(kps)
    end
    local PRESETS = {
        Snow = {
            { size=Vec(22,4,22), oy=5, tex=TX_FLAKE, color=WHITE,
              skeys={{0,1.7,0.35},{1,1.35,0.3}}, squash=-0.12, transp0=0.18,transp1=0.55,
              rate=0.7, speed={0.8,1.6}, life={5,7}, spread=28, rot={-60,60}, rotSpd=26,
              accel=Vec(0,-2.4,0), drag=2.2, glow=0.2, dir=ND.Bottom,
              gust={ax=1.8,az=1.2,wx=0.4,wz=0.31,ph=0.5} },
            { size=Vec(46,4,46), oy=10, tex=TX_SNOWSOFT, color=WHITE,
              skeys={{0,0.48,0.15},{1,0.38,0.10}}, squash=-0.05, transp0=0.7,transp1=0.92,
              rate=12, speed={1.5,3}, life={5,8}, spread=45, rot={-180,180}, rotSpd=10,
              accel=Vec(0,-3.5,0), drag=1.8, glow=0.12, dir=ND.Bottom,
              gust={ax=2.5,az=1.6,wx=0.37,wz=0.29,ph=0} },
            { size=Vec(140,4,140), oy=20, tex=TX_FLAKE, color=Color3.fromRGB(242,248,255),
              skeys={{0,0.5,0.2},{1,0.38,0.14}}, squash=-0.08, transp0=0.08,transp1=0.5,
              rate=190, speed={2,4}, life={7,10}, spread=50, rot={-180,180}, rotSpd=42,
              accel=Vec(0,-5,0), drag=1.7, glow=0.35, dir=ND.Bottom,
              gust={ax=2.2,az=1.4,wx=0.33,wz=0.26,ph=1.9} },
            { size=Vec(220,4,220), oy=32, tex=TX_SNOWSOFT, color=Color3.fromRGB(214,231,255),
              skeys={{0,0.3,0.08},{1,0.22,0.06}}, transp0=0.5,transp1=0.85,
              rate=265, speed={2,4}, life={8,11}, spread=55, rot={-30,30}, rotSpd=12,
              accel=Vec(0,-5.5,0), drag=1.5, glow=0.3, dir=ND.Bottom,
              gust={ax=1.5,az=1,wx=0.29,wz=0.25,ph=3.8} },
            { size=Vec(100,4,100), oy=12, tex=TX_SPARK, color=WHITE,
              skeys={{0,0.4,0.12},{1,0.32,0.1}},
              tkeys={{0,1},{0.2,0.5},{0.45,0.78},{0.62,0.42},{0.85,0.78},{1,1}},
              rate=10, speed={2,4}, life={5,8}, spread=45, rot={-180,180}, rotSpd=60,
              accel=Vec(0,-4.5,0), drag=1.6, glow=0.7, dir=ND.Bottom,
              gust={ax=2.2,az=1.4,wx=0.35,wz=0.27,ph=1} },
        },
        Mist = {
            { size=Vec(150,8,150), oy=0, tex=TX_SOFT, color=Color3.fromRGB(210,216,226),
              size0=26,size1=44, transp0=0.68,transp1=0.94,
              rate=16, speed={0.8,2.2}, life={9,13}, spread=20, rot={-5,5}, rotSpd=3,
              accel=Vec(2.2,0.25,1.2), drag=0.8, dir=ND.Top },
            { size=Vec(110,6,110), oy=1, tex=TX_SOFT, color=Color3.fromRGB(228,232,240),
              size0=12,size1=22, transp0=0.75,transp1=0.95,
              rate=10, speed={1.5,3}, life={6,9}, spread=25, rot={-8,8}, rotSpd=5,
              accel=Vec(3,0.4,1.6), drag=0.8, dir=ND.Top },
        },
        Embers = {
            { size=Vec(90,28,90), oy=2, tex=TX_SOFT, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,   Color3.fromRGB(255,170,60)),
                  ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,95,25)),
                  ColorSequenceKeypoint.new(1,   Color3.fromRGB(165,35,12)) }),
              size0=0.85,size1=0.3, transp0=0.05,transp1=0.85,
              rate=80, speed={3,7}, life={3,5}, spread=40, rot={-60,60}, rotSpd=90,
              accel=Vec(1.5,9,1), drag=1.6, glow=0.35, dir=ND.Top },
            { size=Vec(120,30,120), oy=6, tex=TX_SPARK, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(255,220,140)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(255,110,40)) }),
              size0=0.3,size1=0.08, transp0=0.05,transp1=0.9,
              rate=55, speed={2,5}, life={2.5,4}, spread=55, rot={-40,40}, rotSpd=60,
              accel=Vec(1,7,0.8), drag=1.4, glow=0.9, dir=ND.Top },
        },
        Fireflies = {
            { size=Vec(70,8,70), oy=-2, tex=TX_SNOWSOFT, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,   Color3.fromRGB(220,255,130)),
                  ColorSequenceKeypoint.new(0.5, Color3.fromRGB(205,245,100)),
                  ColorSequenceKeypoint.new(1,   Color3.fromRGB(255,200,80)) }),
              skeys={{0,0.1},{0.08,0.62,0.15},{0.45,0.5,0.1},{1,0.08}},
              tkeys={{0,1},{0.08,0.08},{0.4,0.45},{0.75,0.85},{1,1}},
              rate=18, speed={0.5,1.6}, life={2.5,4}, spread=180, rot={0,0}, rotSpd=0,
              accel=Vec(0,0.3,0), drag=2.5, glow=0.95, dir=ND.Top },
            { size=Vec(140,14,140), oy=-1, tex=TX_SNOWSOFT, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(175,225,95)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(200,190,70)) }),
              skeys={{0,0.06},{0.1,0.32,0.08},{0.5,0.26},{1,0.05}},
              tkeys={{0,1},{0.1,0.3},{0.5,0.6},{1,1}},
              rate=18, speed={0.4,1.2}, life={3,5}, spread=180, rot={0,0}, rotSpd=0,
              accel=Vec(0,0.3,0), drag=2.5, glow=0.95, dir=ND.Top },
            { size=Vec(120,4,120), oy=-4, tex=TX_PUFF, color=Color3.fromRGB(148,158,178),
              size0=13,size1=22, tkeys={{0,1},{0.2,0.8},{0.7,0.89},{1,1}},
              rate=8, speed={0.5,1.5}, life={8,12}, spread=14, rot={-180,180}, rotSpd=4,
              accel=Vec(1.2,0.12,0.7), drag=0.7, glow=0, dir=ND.Top,
              gust={ax=1.2,az=0.8,wx=0.23,wz=0.19,ph=2.2} },
        },
        Petals = {
            { size=Vec(26,6,26), oy=3, tex=TX_SNOWSOFT, zoff=3, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(255,222,232)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(250,205,220)) }),
              skeys={{0,1.75,0.5},{1,1.4,0.3}}, transp0=0.78,transp1=0.92,
              rate=7, speed={0.4,1.1}, life={5,7}, spread=30, rot={-180,180}, rotSpd=22,
              accel=Vec(0,-3,0), drag=1.7, glow=0.04, dir=ND.Bottom,
              gust={ax=2.2,az=1.5,wx=0.43,wz=0.33,ph=1.4} },
            { size=Vec(64,12,64), oy=5, tex=TX_PETAL, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,    Color3.fromRGB(255,222,234)),
                  ColorSequenceKeypoint.new(0.55, Color3.fromRGB(248,178,206)),
                  ColorSequenceKeypoint.new(1,    Color3.fromRGB(226,132,176)) }),
              skeys={{0,2.05,0.5},{1,1.7,0.35}}, qseq=flutterSeq(-0.88, 0.08, 14, 0.12, 0),
              transp0=0.1,transp1=0.38,
              rate=13, speed={0.6,1.4}, life={6,8}, spread=35, rot={-180,180}, rotSpd=95,
              accel=Vec(0,-3.5,0), drag=1.6, glow=0.08, dir=ND.Bottom,
              gust={ax=2.6,az=1.8,wx=0.4,wz=0.31,ph=0.5} },
            { size=Vec(76,12,76), oy=7, tex=TX_PETAL, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,    Color3.fromRGB(250,224,200)),
                  ColorSequenceKeypoint.new(0.55, Color3.fromRGB(244,203,178)),
                  ColorSequenceKeypoint.new(1,    Color3.fromRGB(230,176,150)) }),
              skeys={{0,1.55,0.4},{1,1.28,0.28}}, qseq=flutterSeq(-0.8, 0.15, 12, 0.15, 1),
              transp0=0.14,transp1=0.44,
              rate=12, speed={0.8,1.8}, life={6,8.5}, spread=42, rot={-180,180}, rotSpd=140,
              accel=Vec(0,-3.2,0), drag=1.55, glow=0.05, dir=ND.Bottom,
              gust={ax=3,az=2,wx=0.37,wz=0.29,ph=2.1} },
            { size=Vec(130,6,130), oy=15, tex=TX_PETAL, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(255,222,234)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(246,178,204)) }),
              skeys={{0,0.85,0.26},{1,0.68,0.18}}, qseq=flutterSeq(-0.7, 0.1, 8, 0.2, 0),
              transp0=0.06,transp1=0.44,
              rate=120, speed={1.4,3}, life={7,9}, spread=48, rot={-180,180}, rotSpd=170,
              accel=Vec(0,-5.2,0), drag=1.5, glow=0.1, dir=ND.Bottom,
              gust={ax=3.2,az=2.2,wx=0.33,wz=0.26,ph=1.9} },
            { size=Vec(240,6,240), oy=26, tex=TX_PETAL, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(250,214,228)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(238,190,210)) }),
              skeys={{0,0.3,0.09},{1,0.24,0.06}}, squash=-0.5, transp0=0.42,transp1=0.8,
              rate=170, speed={1.5,3.2}, life={8,11}, spread=55, rot={-180,180}, rotSpd=90,
              accel=Vec(0,-6,0), drag=1.35, glow=0.08, dir=ND.Bottom,
              gust={ax=2.4,az=1.6,wx=0.29,wz=0.25,ph=3.8} },
            { size=Vec(56,1.2,56), oy=-3.7, tex=TX_PETAL, tag="settle",
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(255,214,228)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(240,178,204)) }),
              skeys={{0,1.1,0.3},{1,0.95,0.2}}, qseq=flutterSeq(-0.92, -0.2, 10, 0.1, 0),
              transp0=0.05,transp1=0.42,
              rate=34, speed={0.05,0.5}, life={14,20}, spread=90, rot={-180,180}, rotSpd=25,
              accel=Vec(0,-0.15,0), drag=2.6, glow=0.08, dir=ND.Bottom,
              gust={ax=3.6,az=3,wx=0.5,wz=0.44,ph=0.9} },
        },
        Autumn = {
            { size=Vec(20,6,20), oy=13, uw=8, tex=TX_LEAFA, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,    Color3.fromRGB(226,190,128)),
                  ColorSequenceKeypoint.new(0.45, Color3.fromRGB(255,246,200)),
                  ColorSequenceKeypoint.new(1,    Color3.fromRGB(190,132,62)) }),
              skeys={{0,0.62,0.14},{1,0.54,0.10}}, qseq=flutterSeq(-0.60, 0.14, 13, 0.10, 0),
              transp0=0.08,transp1=0.42,
              rate=14, speed={0.6,1.6}, life={7,9}, spread=34, rot={-180,180}, rotSpd=130,
              accel=Vec(0,-5,0), drag=1.5, glow=0.62, dir=ND.Bottom,
              gust={ax=3.4,az=2.2,wx=0.41,wz=0.3,ph=0.5} },
            { size=Vec(22,6,22), oy=15, uw=8, tex=TX_LEAFB, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(156,74,255)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(106,46,200)) }),
              skeys={{0,0.60,0.14},{1,0.52,0.10}}, qseq=flutterSeq(-0.56, 0.20, 11, 0.14, 1),
              transp0=0.12,transp1=0.46,
              rate=9, speed={0.8,1.9}, life={7,9}, spread=40, rot={-180,180}, rotSpd=185,
              accel=Vec(0,-5.6,0), drag=1.5, glow=0.07, dir=ND.Bottom,
              gust={ax=3.9,az=2.6,wx=0.36,wz=0.27,ph=2.1} },
            { size=Vec(150,8,150), oy=16, uw=26, tex=TX_LEAFA, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,    Color3.fromRGB(226,190,128)),
                  ColorSequenceKeypoint.new(0.45, Color3.fromRGB(255,246,200)),
                  ColorSequenceKeypoint.new(1,    Color3.fromRGB(190,132,62)) }),
              skeys={{0,0.56,0.16},{1,0.44,0.11}}, qseq=flutterSeq(-0.50, 0.10, 8, 0.16, 0),
              transp0=0.08,transp1=0.50,
              rate=24, speed={1.4,3}, life={7,9}, spread=54, rot={-180,180}, rotSpd=210,
              accel=Vec(0,-7,0), drag=1.45, glow=0.62, dir=ND.Bottom,
              gust={ax=4.2,az=2.8,wx=0.33,wz=0.24,ph=1.9} },
            { size=Vec(150,8,150), oy=16, uw=26, tex=TX_LEAFB, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(160,80,52)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(104,44,30)) }),
              skeys={{0,0.54,0.16},{1,0.42,0.11}}, qseq=flutterSeq(-0.54, 0.14, 9, 0.16, 1),
              transp0=0.10,transp1=0.50,
              rate=21, speed={1.4,3.1}, life={7,9}, spread=54, rot={-180,180}, rotSpd=200,
              accel=Vec(0,-7.2,0), drag=1.45, glow=0.07, dir=ND.Bottom,
              gust={ax=4,az=2.7,wx=0.31,wz=0.26,ph=3.3} },
            { size=Vec(150,8,150), oy=16, uw=26, tex=TX_LEAFA, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,   Color3.fromRGB(230,222,176)),
                  ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,254,228)),
                  ColorSequenceKeypoint.new(1,   Color3.fromRGB(200,180,122)) }),
              skeys={{0,0.52,0.15},{1,0.41,0.10}}, qseq=flutterSeq(-0.52, 0.12, 10, 0.16, 1),
              transp0=0.10,transp1=0.52,
              rate=17, speed={1.3,2.9}, life={7,9}, spread=52, rot={-180,180}, rotSpd=190,
              accel=Vec(0,-6.8,0), drag=1.45, glow=0.60, dir=ND.Bottom,
              gust={ax=4.1,az=2.7,wx=0.35,wz=0.28,ph=2.7} },
            { size=Vec(150,8,150), oy=16, uw=26, tex=TX_LEAFB, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(156,74,255)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(106,46,200)) }),
              skeys={{0,0.53,0.16},{1,0.42,0.11}}, qseq=flutterSeq(-0.52, 0.16, 9, 0.16, 0),
              transp0=0.10,transp1=0.50,
              rate=12, speed={1.4,3.1}, life={7,9}, spread=54, rot={-180,180}, rotSpd=220,
              accel=Vec(0,-7.1,0), drag=1.45, glow=0.07, dir=ND.Bottom,
              gust={ax=4,az=2.7,wx=0.3,wz=0.25,ph=5.1} },
            { size=Vec(150,8,150), oy=16, uw=20, tex=TX_LEAFC, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(196,156,112)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(146,110,74)) }),
              skeys={{0,0.50,0.15},{1,0.40,0.10}}, qseq=flutterSeq(-0.52, 0.10, 7, 0.16, 0),
              transp0=0.14,transp1=0.55,
              rate=11, speed={1.5,3.2}, life={7,9}, spread=56, rot={-180,180}, rotSpd=240,
              accel=Vec(0,-7.6,0), drag=1.45, glow=0.08, dir=ND.Bottom,
              gust={ax=3.8,az=2.6,wx=0.29,wz=0.23,ph=0.9} },
            { size=Vec(150,8,150), oy=16, uw=26, tex=TX_LEAFA, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(152,255,255)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(118,214,240)) }),
              skeys={{0,0.52,0.15},{1,0.41,0.10}}, qseq=flutterSeq(-0.50, 0.14, 8, 0.16, 1),
              transp0=0.12,transp1=0.52,
              rate=10, speed={1.4,3}, life={7,9}, spread=54, rot={-180,180}, rotSpd=195,
              accel=Vec(0,-7,0), drag=1.45, glow=0.16, dir=ND.Bottom,
              gust={ax=4.1,az=2.7,wx=0.34,wz=0.27,ph=4.2} },
            { size=Vec(300,30,300), oy=30, uw=60, tex=TX_LEAFA, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,    Color3.fromRGB(226,190,128)),
                  ColorSequenceKeypoint.new(0.45, Color3.fromRGB(255,246,200)),
                  ColorSequenceKeypoint.new(1,    Color3.fromRGB(190,132,62)) }),
              skeys={{0,0.24,0.06},{1,0.18,0.04}}, squash=-0.35, transp0=0.68,transp1=0.92,
              rate=30, speed={1.5,3.2}, life={8,11}, spread=55, rot={-180,180}, rotSpd=110,
              accel=Vec(0,-8,0), drag=1.35, glow=0.55, dir=ND.Bottom,
              gust={ax=3.1,az=2.1,wx=0.27,wz=0.24,ph=1.2} },
            { size=Vec(300,30,300), oy=30, uw=60, tex=TX_LEAFB, color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(160,80,52)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(104,44,30)) }),
              skeys={{0,0.23,0.06},{1,0.17,0.04}}, squash=-0.35, transp0=0.70,transp1=0.93,
              rate=24, speed={1.5,3.2}, life={8,11}, spread=55, rot={-180,180}, rotSpd=120,
              accel=Vec(0,-8,0), drag=1.35, glow=0.07, dir=ND.Bottom,
              gust={ax=3,az=2,wx=0.29,wz=0.22,ph=3.8} },
            { size=Vec(120,16,120), oy=6, tex=TX_LEAFA, tag="streak", aim=true,
              orient=Enum.ParticleOrientation.VelocityParallel,
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,    Color3.fromRGB(226,190,128)),
                  ColorSequenceKeypoint.new(0.45, Color3.fromRGB(255,246,200)),
                  ColorSequenceKeypoint.new(1,    Color3.fromRGB(190,132,62)) }),
              skeys={{0,0.66,0.16},{1,0.46,0.10}}, squash=-0.55,
              tkeys={{0,1},{0.08,0.06},{0.72,0.34},{1,1}},
              rate=0, speed={16,26}, life={1,1.9}, spread=13, rot={-180,180}, rotSpd=300,
              accel=Vec(0,-3.4,0), drag=2.6, glow=0.62, dir=ND.Front,
              gust={ax=5,az=4,wx=0.4,wz=0.35,ph=0} },
            { size=Vec(120,16,120), oy=5, tex=TX_LEAFB, tag="streak", aim=true,
              orient=Enum.ParticleOrientation.VelocityParallel,
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(160,80,52)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(104,44,30)) }),
              skeys={{0,0.62,0.16},{1,0.44,0.10}}, squash=-0.55,
              tkeys={{0,1},{0.08,0.08},{0.72,0.38},{1,1}},
              rate=0, speed={15,24}, life={1,1.8}, spread=14, rot={-180,180}, rotSpd=280,
              accel=Vec(0,-3.6,0), drag=2.6, glow=0.07, dir=ND.Front,
              gust={ax=5,az=4,wx=0.37,wz=0.33,ph=1.7} },
            { size=Vec(96,0.9,96), oy=-3.9, tex=TX_LEAFA, tag="settle",
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0,   Color3.fromRGB(200,150,86)),
                  ColorSequenceKeypoint.new(0.5, Color3.fromRGB(168,110,58)),
                  ColorSequenceKeypoint.new(1,   Color3.fromRGB(130,80,44)) }),
              skeys={{0,0.50,0.14},{1,0.44,0.10}}, qseq=flutterSeq(-0.62, -0.05, 10, 0.1, 0),
              transp0=0.06,transp1=0.46,
              rate=34, speed={0.8,3.2}, life={10,16}, spread=90, rot={-180,180}, rotSpd=90,
              accel=Vec(0,-0.15,0), drag=3.2, glow=0.32, dir=ND.Bottom,
              gust={ax=9,az=8,wx=0.5,wz=0.44,ph=0.9} },
            { size=Vec(70,0.8,70), oy=-3.85, tex=TX_LEAFB, tag="settle",
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(160,80,52)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(104,44,30)) }),
              skeys={{0,0.54,0.12},{1,0.46,0.08}}, qseq=flutterSeq(-0.66, 0.26, 16, 0.06, 0),
              transp0=0.05,transp1=0.44,
              rate=14, speed={0.6,2.2}, life={3,5}, spread=70, rot={-180,180}, rotSpd=340,
              accel=Vec(0,-1.2,0), drag=1.9, glow=0.10, dir=ND.Top,
              gust={ax=8,az=7,wx=0.62,wz=0.55,ph=2.4} },
            { size=Vec(74,0.6,74), oy=-3.95, tex=TX_LEAFC, tag="settle",
              orient=Enum.ParticleOrientation.VelocityPerpendicular,
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(190,146,96)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(150,110,72)) }),
              skeys={{0,0.76,0.16},{1,0.72,0.12}}, squash=0, transp0=0.04,transp1=0.30,
              rate=42, speed={0.04,0.1}, life={20,28}, spread=90, rot={-180,180}, rotSpd=3,
              accel=Vec(0,-0.12,0), drag=4, glow=0.22, dir=ND.Bottom },
            { size=Vec(80,0.8,80), oy=-4, tex=TX_DUST, tag="dust",
              color=ColorSequence.new({
                  ColorSequenceKeypoint.new(0, Color3.fromRGB(206,180,140)),
                  ColorSequenceKeypoint.new(1, Color3.fromRGB(150,124,92)) }),
              skeys={{0,1.1,0.35},{1,3.4,0.5}}, tkeys={{0,1},{0.18,0.84},{0.7,0.93},{1,1}},
              rate=0, speed={2,6}, life={1.1,2}, spread=85, rot={-180,180}, rotSpd=22,
              accel=Vec(0,0.6,0), drag=2.4, glow=0.06, dir=ND.Top,
              gust={ax=8,az=7,wx=0.4,wz=0.35,ph=0} },
        },
        Ash = {
            { size=Vec(110,4,110), oy=16, tex=TX_SOFT, color=Color3.fromRGB(96,90,86),
              size0=0.4,size1=0.3, transp0=0.05,transp1=0.5,
              rate=140, speed={2,4}, life={6,9}, spread=45, rot={-60,60}, rotSpd=45,
              accel=Vec(-3.5,-5,2), drag=1.6, glow=0, dir=ND.Bottom },
            { size=Vec(170,4,170), oy=26, tex=TX_SPARK, color=Color3.fromRGB(255,140,70),
              size0=0.14,size1=0.05, transp0=0.15,transp1=0.9,
              rate=26, speed={1.5,3.5}, life={4,7}, spread=60, rot={-40,40}, rotSpd=70,
              accel=Vec(-2.5,-2,1.5), drag=1.5, glow=0.9, dir=ND.Bottom },
        },
        Sandstorm = {
            { size=Vec(160,20,160), oy=6, tex=TX_SOFT, color=Color3.fromRGB(194,168,120),
              size0=24,size1=38, transp0=0.58,transp1=0.93,
              rate=24, speed={8,15}, life={5,8}, spread=28, rot={-8,8}, rotSpd=6,
              accel=Vec(20,0.5,7), drag=0.6, dir=ND.Right },
        },
    }
    Weather.TypeOrder = { "Rain", "Snow", "Mist", "Embers", "Fireflies", "Petals", "Autumn",
                          "Ash", "Sandstorm", "BloodMoon" }
    local MOOD = {
        Rain      = { B=-0.03, C=0.08,  S=-0.12, tint=Color3.fromRGB(205,220,255) },
        Snow      = { B= 0.03, C=0.05,  S=-0.08, tint=Color3.fromRGB(226,240,255) },
        Mist      = { B=-0.01, C=-0.05, S=-0.20, tint=Color3.fromRGB(220,224,230) },
        Embers    = { B= 0.02, C=0.08,  S= 0.10, tint=Color3.fromRGB(255,238,220) },
        Fireflies = { B=-0.04, C=0.06,  S= 0.02, tint=Color3.fromRGB(238,228,205) },
        Petals    = { B= 0.00, C=0.09,  S= 0.05, tint=Color3.fromRGB(255,234,234) },
        Autumn    = { B= 0.005, C=0.08, S= 0.04, tint=Color3.fromRGB(253,244,232) },
        Ash       = { B=-0.04, C=0.06,  S=-0.25, tint=Color3.fromRGB(225,220,215) },
        Sandstorm = { B=-0.02, C=0.10,  S= 0.05, tint=Color3.fromRGB(224,196,150) },
        BloodMoon = { B=-0.05, C=0.12,  S=-0.20, tint=Color3.fromRGB(255,180,180) },
    }
    local SOUND = { Rain="rain", Snow="wind", Mist="wind", Embers="fire",
                    Fireflies="night", Petals="birds", Autumn="wind", Ash="wind",
                    Sandstorm="wind", BloodMoon="night" }
    local function ambientId(mood)
        local map = cfg("WeatherSoundIds", nil)
        if type(map) == "table" and map[mood] and map[mood] ~= "" then return map[mood] end
        return nil
    end
    local _partLayers = {}
    local _ambient  = nil
    local _moodCC   = nil
    local _moodTween = nil
    local FADE_TI = TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
    local MOOD_TI = TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
    local _followConn = nil
    local _lightFolder = nil
    local _rays = nil
    local _special = { moon = nil, shell = nil, sky = nil, cc = nil, atmo = nil, halo = nil, kind = nil }
    local _specialConn = nil
    local _clockConn = nil
    local function clearPartLayers()
        for _, L in _partLayers do
            if L.rateTween then L.rateTween:Cancel() end
            if L.host then pcall(function() L.host:Destroy() end) end
        end
        table.clear(_partLayers)
    end
    local _fadeLayers = {}
    local _fadeToken = 0
    local function killFade()
        _fadeToken = _fadeToken + 1
        for _, F in _fadeLayers do
            local host = F.host
            pcall(function() host:Destroy() end)
        end
        table.clear(_fadeLayers)
    end
    local function beginLayerFade()
        killFade()
        local maxLife = 0
        for _, L in _partLayers do
            if L.rateTween then L.rateTween:Cancel() end
            if L.emitter then
                TweenService:Create(L.emitter, FADE_TI, { Rate = 0 }):Play()
            end
            if L.life2 > maxLife then maxLife = L.life2 end
            _fadeLayers[#_fadeLayers + 1] = { host = L.host, oy = L.oy }
        end
        table.clear(_partLayers)
        local tok = _fadeToken
        task.delay(FADE_TI.Time + maxLife, function()
            if tok == _fadeToken then killFade() end
        end)
    end
    local _spectacles, _spectWeight, _spectNextT = {}, 0, 0
    local function registerSpectacle(name, weight, fn)
        _spectacles[#_spectacles + 1] = { name = name, weight = weight, fn = fn }
        _spectWeight = _spectWeight + weight
    end
    local _blizzT0, _blizzDur = 0, 1
    local function layerRate(L, I)
        if L.spec.tag == "settle" then
            return L.spec.rate * iRate(I, SET_LO, SET_HI)
        end
        return L.spec.rate * iRate(I, L.ir.lo, L.ir.hi)
    end
    local function tuneLayer(L, I)
        local spec, R, e = L.spec, L.ir, L.emitter
        local am = 1
        if spec.tag ~= "settle" then
            am = 1 - R.acc + R.acc * I
        end
        L.accel = spec.accel * am
        if not spec.gust then
            e.Acceleration = L.accel
        end
        e.Speed = NumberRange.new(spec.speed[1] * am, spec.speed[2] * am)
        if spec.gust then
            local gm = 1 - R.gst + R.gst * I
            L.gax, L.gaz = spec.gust.ax * gm, spec.gust.az * gm
        end
        local sm = 1 - R.sz + R.sz * I
        if spec.skeys then
            local ks = spec.skeys
            local kps = table.create(#ks)
            for i = 1, #ks do
                local k = ks[i]
                kps[i] = NumberSequenceKeypoint.new(k[1], k[2] * sm, (k[3] or 0) * sm)
            end
            e.Size = NumberSequence.new(kps)
        else
            e.Size = NumberSequence.new(spec.size0 * sm, spec.size1 * sm)
        end
        if not spec.tkeys then
            local om = 1 - R.alp + R.alp * I
            e.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0,    1),
                NumberSequenceKeypoint.new(0.12, math.clamp(1 - (1 - spec.transp0) * om, 0, 1)),
                NumberSequenceKeypoint.new(0.8,  math.clamp(1 - (1 - spec.transp1) * om, 0, 1)),
                NumberSequenceKeypoint.new(1,    1),
            })
        end
    end
    local function mkPartLayer(spec, ir, rampIn)
        local host = Instance.new("Part")
        host.Name = "_wxHost"; host.Anchored = true; host.CanCollide = false; host.CanQuery = false
        host.CanTouch = false; host.CastShadow = false; host.Transparency = 1; host.Massless = true
        host.Size = spec.size; host:SetAttribute("WX_Custom", true)
        host.Parent = getFolder()
        local e = Instance.new("ParticleEmitter")
        e:SetAttribute("WX_Custom", true)
        e.Texture = spec.tex
        local colorSeq = spec.color
        if typeof(colorSeq) ~= "ColorSequence" then
            colorSeq = ColorSequence.new(colorSeq)
        end
        e.Color = colorSeq
        local glow = spec.glow or 0
        e.LightEmission  = glow
        e.LightInfluence = 1 - math.min(glow, 1)
        e.Lifetime = NumberRange.new(spec.life[1], spec.life[2])
        e.SpreadAngle = Vector2.new(spec.spread, spec.spread)
        local rot0, rot1 = 0, 360
        if spec.rot then
            rot0, rot1 = spec.rot[1], spec.rot[2]
        end
        e.Rotation = NumberRange.new(rot0, rot1)
        e.RotSpeed = NumberRange.new(-(spec.rotSpd or 45), spec.rotSpd or 45)
        if spec.qseq then
            e.Squash = spec.qseq
        else
            e.Squash = NumberSequence.new(spec.squash or 0)
        end
        if spec.tkeys then
            local kps = table.create(#spec.tkeys)
            for i = 1, #spec.tkeys do kps[i] = NumberSequenceKeypoint.new(spec.tkeys[i][1], spec.tkeys[i][2]) end
            e.Transparency = NumberSequence.new(kps)
        end
        e.Drag = spec.drag or 0
        e.ZOffset = spec.zoff or 0
        e.EmissionDirection = spec.dir
        if spec.orient then
            e.Orientation = spec.orient
        end
        e.Parent = host
        local L = { host = host, oy = spec.oy, emitter = e, spec = spec, ir = ir,
                    accel = spec.accel, gust = spec.gust, tag = spec.tag,
                    gax = 0, gaz = 0, rateMul = 1,
                    uw = spec.uw, aim = spec.aim,
                    life2 = spec.life[2], rateTween = nil }
        local I = iAmt()
        tuneLayer(L, I)
        local targetRate = layerRate(L, I)
        if rampIn then
            e.Rate = 0
            L.rateTween = TweenService:Create(e, FADE_TI, { Rate = targetRate })
            L.rateTween:Play()
        else
            e.Rate = targetRate
        end
        _partLayers[#_partLayers + 1] = L
    end
    local function clearMood()
        if _moodTween then _moodTween:Cancel(); _moodTween = nil end
        if _moodCC then pcall(function() _moodCC:Destroy() end); _moodCC = nil end
    end
    local function applyMood(name)
        if not cfg("WeatherMood", true) then
            clearMood()
            return
        end
        local m = MOOD[name]
        if not m then
            clearMood()
            return
        end
        if _moodTween then _moodTween:Cancel(); _moodTween = nil end
        local g = 0.6 + 0.4 * iAmt()
        local cc = _moodCC
        if cc and cc.Parent then
            _moodTween = TweenService:Create(cc, MOOD_TI,
                { Brightness = m.B * g, Contrast = m.C * g, Saturation = m.S * g, TintColor = m.tint })
            _moodTween:Play()
            return
        end
        cc = Instance.new("ColorCorrectionEffect")
        cc.Name = "_wxMood"; cc:SetAttribute("WX_Custom", true)
        cc.Brightness = m.B * g; cc.Contrast = m.C * g; cc.Saturation = m.S * g; cc.TintColor = m.tint
        cc.Parent = Lighting
        _moodCC = cc
    end
    local function startAmbient(mood)
        if _ambient then pcall(function() _ambient:Destroy() end); _ambient = nil end
        local id = mood and ambientId(mood)
        if not id then return end
        local s = Instance.new("Sound")
        s.Name = "_wxAmb"; s:SetAttribute("WX_Custom", true)
        s.SoundId = id; s.Looped = true; s.Volume = cfg("WeatherSoundVolume", 0.35)
        s.Parent = SoundService
        pcall(function() s:Play() end)
        _ambient = s
    end
    local function oneShot(id, vol, pitch)
        if not id or id == "" then return end
        local s = Instance.new("Sound")
        s.Name = "_wxSfx"; s:SetAttribute("WX_Custom", true)
        s.SoundId = id; s.Volume = math.clamp(vol, 0, 10)
        if pitch then s.PlaybackSpeed = pitch end
        s.Parent = SoundService
        pcall(function() s:Play() end)
        Debris:AddItem(s, 8)
    end
    local function getLightFolder()
        if _lightFolder and _lightFolder.Parent then return _lightFolder end
        local f = Instance.new("Folder"); f.Name = "_wxBolts"; f:SetAttribute("WX_Custom", true)
        f.Parent = getFolder()
        _lightFolder = f; return f
    end
    local function mkBoltPart(color, transp, size, cf, parent)
        local p = Instance.new("Part")
        p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false
        p.CastShadow = false; p.Massless = true; p.Material = Enum.Material.Neon
        p.Color = color; p.Transparency = transp; p:SetAttribute("WX_Custom", true)
        p.Size = size; p.CFrame = cf; p.Parent = parent
        return p
    end
    local startStorm, stopStormLoop, stopStorm, rescheduleStorm
    ;(function()
        local TX_BGLOW   = "rbxassetid://78582616787441"
        local TX_BSTREAK = "rbxassetid://102481842205398"
        local BOLT_CORE  = Color3.fromRGB(245, 242, 255)
        local BOLT_GLOW  = Color3.fromRGB(162, 145, 255)
        local BOLT_AFTER = Color3.fromRGB(126, 96, 228)
        local ROT90 = CFrame.Angles(0, math.rad(90), 0)
        local FLICK = 0.12
        local _stormConn, _stormNextT = nil, 0
        local _boltRp = RaycastParams.new()
        _boltRp.FilterType = Enum.RaycastFilterType.Exclude
        local function mkSeg(color, transp, dia, len, cf, parent)
            local p = Instance.new("Part")
            p.Shape = Enum.PartType.Cylinder
            p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false
            p.CastShadow = false; p.Massless = true; p.Material = Enum.Material.Neon
            p.Color = color; p.Transparency = transp; p:SetAttribute("WX_Custom", true)
            p.Size = Vec(len, dia, dia); p.CFrame = cf * ROT90; p.Parent = parent
            return p
        end
        local function boltChannel(model, segs, top, bot, n, thick, jitter, layer3)
            local pts = { top }
            local prev = top
            local driftAng = math.random() * 6.283
            for i = 1, n do
                local t = i / n
                local point = top:Lerp(bot, t)
                if i < n then
                    driftAng = driftAng + (math.random() - 0.5) * 2.2
                    local j = jitter * (0.5 + 0.5 * (1 - t))
                    local off = j * (0.3 + math.random() * 0.7)
                    point = point + Vec(math.cos(driftAng) * off, (math.random() - 0.5) * j * 0.25, math.sin(driftAng) * off)
                end
                local len = math.max((point - prev).Magnitude, 0.1)
                local cf = CFrame.lookAt((point + prev) * 0.5, point)
                segs[#segs + 1] = { p = mkSeg(BOLT_CORE, 0.02, thick, len + thick * 1.6, cf, model), on = 0.02, fade = 0.16 }
                segs[#segs + 1] = { p = mkSeg(BOLT_GLOW, 0.78, thick * 5.5, len + thick * 1.8, cf, model), on = 0.78, fade = 0.38 }
                if layer3 then
                    segs[#segs + 1] = { p = mkSeg(BOLT_AFTER, 0.88, thick * 2.2, len + thick * 1.6, cf, model), on = 0.75, fade = 0.85 }
                end
                pts[#pts + 1] = point
                prev = point
            end
            return pts
        end
        local function boltBranch(model, segs, fromPts, groundY, thick, depth)
            local count = 1
            if depth == 1 then count = math.random(2, 4) end
            for _ = 1, count do
                local idx = math.random(math.floor(#fromPts * 0.2), math.floor(#fromPts * 0.7))
                local a = fromPts[math.max(idx, 2)]
                local ang = math.random() * 6.283
                local drop = (a.Y - groundY) * (0.3 + math.random() * 0.25)
                local outw = drop * (0.55 + math.random() * 0.4)
                local endp = a + Vec(math.cos(ang) * outw, -drop, math.sin(ang) * outw)
                local n = 5
                if depth == 1 then n = 8 end
                local blen = (endp - a).Magnitude
                local pts = boltChannel(model, segs, a, endp, n, thick, math.max(5, blen * 0.1) / depth, false)
                if depth < 3 and math.random() < 0.5 then
                    boltBranch(model, segs, pts, groundY, thick * 0.45, depth + 1)
                end
            end
        end
        local function preFlash(pos, scale)
            local host = mkBoltPart(BOLT_CORE, 1, Vec(2, 2, 2), CFrame.new(pos), getLightFolder())
            local e = Instance.new("ParticleEmitter")
            e:SetAttribute("WX_Custom", true)
            e.Texture = TX_BGLOW
            e.Color = ColorSequence.new(Color3.fromRGB(206, 192, 255))
            e.LightEmission = 0.85; e.LightInfluence = 0
            e.Size = NumberSequence.new(60 * scale, 85 * scale)
            e.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(0.25, 0.55),
                NumberSequenceKeypoint.new(0.7, 0.75), NumberSequenceKeypoint.new(1, 1),
            })
            e.Lifetime = NumberRange.new(0.22, 0.3)
            e.Rate = 0; e.Speed = NumberRange.new(0, 0.5)
            e.Parent = host
            e:Emit(1)
            task.delay(0.09, function() pcall(function() e:Emit(1) end) end)
            Debris:AddItem(host, 1.2)
        end
        local function screenFlash()
            local cc = Instance.new("ColorCorrectionEffect")
            cc.Name = "_wxFlash"; cc:SetAttribute("WX_Custom", true); cc.Brightness = 0.35; cc.Parent = Lighting
            local pulseA = 0.3 + math.random() * 0.25
            local pulseB = -1
            if math.random() < 0.6 then pulseB = pulseA + 0.18 + math.random() * 0.12 end
            local st = tick(); local c2
            c2 = RunService.Heartbeat:Connect(function()
                if not cc.Parent then if c2 then c2:Disconnect() end return end
                local a = (tick() - st) / 0.45
                if a >= 1 then pcall(function() cc:Destroy() end); if c2 then c2:Disconnect() end return end
                local b = 0.35 * (1 - a)
                if a >= pulseA and a < pulseA + 0.14 then
                    b = b + 0.22 * (1 - (a - pulseA) / 0.14)
                elseif pulseB > 0 and a >= pulseB and a < pulseB + 0.14 then
                    b = b + 0.16 * (1 - (a - pulseB) / 0.14)
                end
                cc.Brightness = b
            end)
            Debris:AddItem(cc, 0.7)
        end
        local function igniteBolt(ground, top)
            local model = Instance.new("Model"); model.Name = "_bolt"; model:SetAttribute("WX_Custom", true)
            local segs = {}
            local mainPts = boltChannel(model, segs, top, ground, 18, 1.0, 14, true)
            boltBranch(model, segs, mainPts, ground.Y, 0.55, 1)
            local dome = mkBoltPart(Color3.fromRGB(228, 240, 255), 0.5, Vec(0.6, 5, 5),
                CFrame.new(ground + Vec(0, 0.3, 0)) * CFrame.Angles(0, 0, math.rad(90)), model)
            dome.Shape = Enum.PartType.Cylinder
            local fl = Instance.new("PointLight"); fl.Color = BOLT_GLOW; fl.Range = 150; fl.Brightness = 8; fl.Parent = dome
            TweenService:Create(dome, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                { Size = Vec(0.6, 30, 30), Transparency = 1 }):Play()
            TweenService:Create(fl, TweenInfo.new(0.5), { Brightness = 0 }):Play()
            local cb = mkBoltPart(BOLT_CORE, 1, Vec(2, 2, 2), CFrame.new(ground + Vec(0, 1.5, 0)), model)
            local ce = Instance.new("ParticleEmitter")
            ce:SetAttribute("WX_Custom", true)
            ce.Texture = TX_BGLOW
            ce.Color = ColorSequence.new(Color3.fromRGB(235, 228, 255))
            ce.LightEmission = 1; ce.LightInfluence = 0
            ce.Size = NumberSequence.new(13, 17)
            ce.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.3), NumberSequenceKeypoint.new(1, 1),
            })
            ce.Lifetime = NumberRange.new(0.3, 0.4)
            ce.Rate = 0; ce.Speed = NumberRange.new(0, 0.1)
            ce.Parent = cb
            local se = Instance.new("ParticleEmitter")
            se:SetAttribute("WX_Custom", true)
            se.Texture = TX_BSTREAK
            se.Color = ColorSequence.new(Color3.fromRGB(220, 205, 255))
            se.LightEmission = 1; se.LightInfluence = 0
            se.Size = NumberSequence.new(2, 3)
            se.Transparency = NumberSequence.new(0.25, 1)
            se.Lifetime = NumberRange.new(0.5, 0.9)
            se.Rate = 0; se.Speed = NumberRange.new(26, 44)
            se.SpreadAngle = Vector2.new(38, 38)
            se.Orientation = Enum.ParticleOrientation.VelocityParallel
            se.EmissionDirection = Enum.NormalId.Top
            se.Acceleration = Vec(0, -40, 0)
            se.Parent = cb
            ce:Emit(1)
            se:Emit(9)
            model.Parent = getLightFolder()
            preFlash(top + Vec(0, 10, 0), 1.5)
            local flickEnd = FLICK
            local restrikeAt = -1
            if math.random() < 0.3 then restrikeAt = FLICK + 0.08 + math.random() * 0.07 end
            local startT = tick(); local conn
            conn = RunService.Heartbeat:Connect(function()
                if not model.Parent then if conn then conn:Disconnect() end return end
                local a = tick() - startT
                if restrikeAt > 0 and a >= restrikeAt then
                    restrikeAt = -1
                    flickEnd = a + 0.1
                    fl.Brightness = 6
                    TweenService:Create(fl, TweenInfo.new(0.4), { Brightness = 0 }):Play()
                end
                if a < flickEnd then
                    local dim = 0
                    if math.random() < 0.28 then dim = 0.85 end
                    for i = 1, #segs do
                        local s = segs[i]
                        s.p.Transparency = s.on + (1 - s.on) * dim
                    end
                    return
                end
                local f = a - flickEnd
                if f >= 0.85 then
                    if conn then conn:Disconnect() end
                    pcall(function() model:Destroy() end)
                    return
                end
                for i = 1, #segs do
                    local s = segs[i]
                    local k = math.clamp(f / s.fade, 0, 1)
                    s.p.Transparency = s.on + (1 - s.on) * k
                end
            end)
            Debris:AddItem(model, 1.8)
            if cfg("WeatherStormFlash", true) then screenFlash() end
            local tid = cfg("WeatherThunderId", "")
            if tid ~= "" then
                local cam = Camera
                local dist = 120
                if cam then dist = (ground - cam.CFrame.Position).Magnitude end
                task.delay(dist * (3 / 343), function()
                    oneShot(tid, cfg("WeatherSoundVolume", 0.35) * 8 * (0.85 + math.random() * 0.3),
                        0.92 + math.random() * 0.16)
                end)
            end
        end
        local function spawnBolt()
            if not Config.Weather or not cfg("WeatherStorm", false) then return end
            local cam = Camera; if not cam then return end
            local base = cam.CFrame.Position
            local lv = cam.CFrame.LookVector
            local flat = Vec(lv.X, 0, lv.Z)
            if flat.Magnitude < 0.05 then flat = Vec(0, 0, -1) else flat = flat.Unit end
            local right = Vec(flat.Z, 0, -flat.X)
            local fwd  = 55 + math.random() * 95
            local side = (math.random() - 0.5) * 85
            local gx = base.X + flat.X * fwd + right.X * side
            local gz = base.Z + flat.Z * fwd + right.Z * side
            local ground = Vec(gx + (math.random() - 0.5) * 20, base.Y - 45, gz + (math.random() - 0.5) * 20)
            local ch = lp and lp.Character
            if ch then
                _boltRp.FilterDescendantsInstances = { getFolder(), ch }
            else
                _boltRp.FilterDescendantsInstances = { getFolder() }
            end
            local hit = Workspace:Raycast(Vec(gx, base.Y + 140, gz), Vec(0, -400, 0), _boltRp)
            if hit then ground = hit.Position end
            local top = Vec(gx + (math.random() - 0.5) * 90, base.Y + 280, gz + (math.random() - 0.5) * 90)
            preFlash(Vec(top.X, top.Y - 20, top.Z), 1)
            task.delay(0.1 + math.random() * 0.2, function()
                if not Config.Weather or not cfg("WeatherStorm", false) then return end
                pcall(function() igniteBolt(ground, top) end)
            end)
        end
        function startStorm()
            if _stormConn then return end
            _stormNextT = tick() + 2 + math.random() * 3
            _stormConn = RunService.Heartbeat:Connect(function()
                if not Config.Weather or not cfg("WeatherStorm", false) then return end
                local now = tick()
                if now >= _stormNextT then
                    _stormNextT = now + cfg("WeatherStormMin", 4)
                        + math.random() * cfg("WeatherStormVar", 8)
                    pcall(spawnBolt)
                end
            end)
        end
        function rescheduleStorm()
            if not _stormConn then
                return
            end
            local cap = tick() + cfg("WeatherStormMin", 4) + cfg("WeatherStormVar", 8)
            _stormNextT = math.min(_stormNextT, cap)
        end
        function stopStormLoop()
            if _stormConn then _stormConn:Disconnect(); _stormConn = nil end
        end
        function stopStorm()
            stopStormLoop()
            if _lightFolder then pcall(function() _lightFolder:Destroy() end); _lightFolder = nil end
        end
        registerSpectacle("StormBarrage", 1, function()
            if not Config.Weather or not cfg("WeatherStorm", false) then return end
            for i = 0, 2 do
                task.delay(i * (0.35 + math.random() * 0.3), function() pcall(spawnBolt) end)
            end
        end)
    end)()
    local function startFollow()
        if _followConn then return end
        local now0 = tick()
        Wind.reset(now0)
        _spectNextT = now0 + 120 + math.random() * 120
        _followConn = RunService.Heartbeat:Connect(function()
            if not Config.Weather then return end
            local now = tick()
            Wind.update(now)
            local amb = _ambient
            if amb then
                local vol = cfg("WeatherSoundVolume", 0.35)
                if amb.Volume ~= vol then amb.Volume = vol end
            end
            local cam = Camera; if not cam then return end
            local pos = cam.CFrame.Position
            if now >= _spectNextT then
                _spectNextT = now + 120 + math.random() * 120
                if _spectWeight > 0 then
                    local r = math.random() * _spectWeight
                    for i = 1, #_spectacles do
                        local s = _spectacles[i]
                        r = r - s.weight
                        if r <= 0 then
                            pcall(s.fn)
                            break
                        end
                    end
                end
            end
            local wgain = 1
            local bt = (now - _blizzT0) / _blizzDur
            if bt >= 0 and bt < 1 then
                wgain = 1 + 1.7 * math.sin(bt * math.pi)
            end
            local nwx, nwz = 1, 0
            local wl = math.sqrt(Wind.x * Wind.x + Wind.z * Wind.z)
            if wl > 0.05 then
                nwx = Wind.x / wl
                nwz = Wind.z / wl
            end
            for i = 1, #_partLayers do
                local L = _partLayers[i]
                if L.host and L.host.Parent then
                    local hx, hz = pos.X, pos.Z
                    if L.uw then
                        hx = hx - nwx * L.uw
                        hz = hz - nwz * L.uw
                    end
                    local hy = pos.Y + L.oy
                    if L.aim then
                        L.host.CFrame = CFrame.lookAt(Vec(hx, hy, hz), Vec(hx + nwx, hy, hz + nwz))
                    else
                        L.host.CFrame = CFrame.new(hx, hy, hz)
                    end
                    local g = L.gust
                    if g then
                        L.emitter.Acceleration = L.accel + Vec(
                            Wind.x * wgain * L.gax * (1 + 0.3 * math.sin(now * g.wx + g.ph)), 0,
                            Wind.z * wgain * L.gaz * (1 + 0.3 * math.cos(now * g.wz + g.ph)))
                    end
                end
            end
            for i = 1, #_fadeLayers do
                local F = _fadeLayers[i]
                if F.host and F.host.Parent then
                    F.host.CFrame = CFrame.new(pos.X, pos.Y + F.oy, pos.Z)
                end
            end
        end)
    end
    local function stopFollow()
        if _followConn then _followConn:Disconnect(); _followConn = nil end
    end
    do
        local SWELL_UP = TweenInfo.new(2.5, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
        local SWELL_DOWN = TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
        local function windSwell(kind, dur, rateMul)
            if not Config.Weather or _curType ~= kind then return end
            _blizzT0, _blizzDur = tick(), dur
            local surged = {}
            local I = iAmt()
            for _, L in _partLayers do
                if L.emitter and L.tag ~= "settle" then
                    if L.rateTween then L.rateTween:Cancel() end
                    L.rateMul = rateMul
                    local tw = TweenService:Create(L.emitter, SWELL_UP, { Rate = layerRate(L, I) * rateMul })
                    L.rateTween = tw
                    tw:Play()
                    surged[#surged + 1] = L
                end
            end
            task.delay(dur - 3, function()
                local I2 = iAmt()
                for _, L in surged do
                    local live = false
                    for _, cur in _partLayers do
                        if cur == L then live = true break end
                    end
                    if live and L.emitter and L.emitter.Parent then
                        if L.rateTween then L.rateTween:Cancel() end
                        L.rateMul = 1
                        local tw = TweenService:Create(L.emitter, SWELL_DOWN, { Rate = layerRate(L, I2) })
                        L.rateTween = tw
                        tw:Play()
                    end
                end
            end)
        end
        registerSpectacle("SnowBlizzard", 1, function()
            windSwell("Snow", 11 + math.random() * 3, 1.9)
        end)
        registerSpectacle("SakuraGale", 1, function()
            windSwell("Petals", 9 + math.random() * 4, 1.7)
        end)
    end
    do
        local GUST_UP   = TweenInfo.new(1.1, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
        local GUST_DOWN = TweenInfo.new(2.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
        local STREAK_BURST = { {0.3, 18}, {0.65, 14}, {1.05, 9} }
        Wind.onGust(function(dur)
            if not Config.Weather or _curType ~= "Autumn" then return end
            local I = iAmt()
            local em = iRate(I, IR.Autumn.lo, IR.Autumn.hi)
            local gm = 1 + 0.4 * I
            local surged = {}
            for _, L in _partLayers do
                if L.emitter and L.tag == "dust" then
                    local e = L.emitter
                    local n2 = math.max(1, math.floor(10 * em + 0.5))
                    e:Emit(math.max(1, math.floor(14 * em + 0.5)))
                    task.delay(0.4, function()
                        if e.Parent then e:Emit(n2) end
                    end)
                elseif L.emitter and L.tag == "streak" then
                    local e = L.emitter
                    e:Emit(math.max(1, math.floor(20 * em + 0.5)))
                    for _, b in STREAK_BURST do
                        local n = math.max(1, math.floor(b[2] * em + 0.5))
                        task.delay(b[1], function()
                            if e.Parent then e:Emit(n) end
                        end)
                    end
                elseif L.emitter and L.tag ~= "settle" then
                    if L.rateTween then L.rateTween:Cancel() end
                    L.rateMul = gm
                    local tw = TweenService:Create(L.emitter, GUST_UP, { Rate = layerRate(L, I) * gm })
                    L.rateTween = tw
                    tw:Play()
                    surged[#surged + 1] = L
                end
            end
            task.delay(dur, function()
                local I2 = iAmt()
                for _, L in surged do
                    local live = false
                    for _, cur in _partLayers do
                        if cur == L then live = true break end
                    end
                    if live and L.emitter and L.emitter.Parent then
                        if L.rateTween then L.rateTween:Cancel() end
                        L.rateMul = 1
                        local tw = TweenService:Create(L.emitter, GUST_DOWN, { Rate = layerRate(L, I2) })
                        L.rateTween = tw
                        tw:Play()
                    end
                end
            end)
        end)
    end
    local function clearSpecial()
        if _specialConn then _specialConn:Disconnect(); _specialConn = nil end
        if _special.moon  then pcall(function() _special.moon:Destroy() end);  _special.moon  = nil end
        if _special.shell then pcall(function() _special.shell:Destroy() end); _special.shell = nil end
        if _special.sky   then pcall(function() _special.sky:Destroy() end);   _special.sky   = nil end
        if _special.cc    then pcall(function() _special.cc:Destroy() end);    _special.cc    = nil end
        if _special.atmo  then pcall(function() _special.atmo:Destroy() end);  _special.atmo  = nil end
        _special.halo = nil
        _special.kind = nil
    end
    local function tuneBloodMoon(I)
        local ms = 0.72 + 0.28 * I
        local gr = 0.5 + 0.5 * I
        if _special.moon then
            _special.moon.Size = Vec(130, 130, 130) * ms
        end
        if _special.shell then
            _special.shell.Size = Vec(160, 160, 160) * ms
        end
        if _special.halo then
            _special.halo.Rate = 25 * iRate(I, 1.45, 1.05)
            _special.halo.Size = NumberSequence.new(330 * ms)
        end
        if _special.cc then
            _special.cc.Saturation = -0.15 * gr
            _special.cc.Brightness = -0.04 * gr
        end
        if _special.atmo then
            _special.atmo.Density = 0.12 + 0.18 * I
            _special.atmo.Haze    = 0.9 + 0.9 * I
            _special.atmo.Glare   = 0.4 + 0.4 * I
        end
        if _special.sky then
            _special.sky.StarCount = math.floor(4000 * (0.55 + 0.45 * I))
        end
    end
    local function buildBloodMoon()
        local folder = getFolder()
        local s = Instance.new("Sky"); s.Name = "_wxSkyBM"; s:SetAttribute("WX_Custom", true)
        s.SkyboxBk = "rbxassetid://159454299"; s.SkyboxDn = "rbxassetid://159454296"
        s.SkyboxFt = "rbxassetid://159454293"; s.SkyboxLf = "rbxassetid://159454286"
        s.SkyboxRt = "rbxassetid://159454300"; s.SkyboxUp = "rbxassetid://159454288"
        s.SunAngularSize = 0; s.MoonAngularSize = 0; s.StarCount = 4000
        s.Parent = Lighting; _special.sky = s
        local moon = Instance.new("Part")
        moon.Shape = Enum.PartType.Ball; moon.Size = Vec(130, 130, 130)
        moon.Material = Enum.Material.Neon; moon.Color = Color3.fromRGB(215, 45, 35)
        moon.Anchored = true; moon.CanCollide = false; moon.CanQuery = false; moon.CanTouch = false
        moon.CastShadow = false; moon.Massless = true; moon:SetAttribute("WX_Custom", true)
        local att = Instance.new("Attachment"); att.Parent = moon
        local halo = Instance.new("ParticleEmitter")
        halo.Texture = TX_SOFT; halo.Color = ColorSequence.new(Color3.fromRGB(255, 60, 42))
        halo.LightEmission = 1; halo.LightInfluence = 0
        halo.Rate = 25; halo.Lifetime = NumberRange.new(0.22, 0.28)
        halo.Speed = NumberRange.new(0, 0); halo.Size = NumberSequence.new(330)
        halo.Transparency = NumberSequence.new(0.72); halo.RotSpeed = NumberRange.new(0, 0)
        halo.Parent = att; _special.halo = halo
        moon.Parent = folder; _special.moon = moon
        local shell = Instance.new("Part")
        shell.Shape = Enum.PartType.Ball; shell.Size = Vec(160, 160, 160)
        shell.Material = Enum.Material.Neon; shell.Color = Color3.fromRGB(255, 60, 45); shell.Transparency = 0.7
        shell.Anchored = true; shell.CanCollide = false; shell.CanQuery = false; shell.CanTouch = false
        shell.CastShadow = false; shell.Massless = true; shell:SetAttribute("WX_Custom", true)
        shell.Parent = folder; _special.shell = shell
        local cc = Instance.new("ColorCorrectionEffect"); cc.Name = "_wxBM"; cc:SetAttribute("WX_Custom", true)
        cc.TintColor = Color3.fromRGB(255, 165, 160); cc.Saturation = -0.15; cc.Brightness = -0.04
        cc.Parent = Lighting; _special.cc = cc
        local at = Instance.new("Atmosphere"); at.Name = "_wxBMAtmo"; at:SetAttribute("WX_Custom", true)
        at.Density = 0.3; at.Color = Color3.fromRGB(125, 35, 35); at.Decay = Color3.fromRGB(190, 55, 50)
        at.Glare = 0.8; at.Haze = 1.8; at.Parent = Lighting; _special.atmo = at
        _special.kind = "BloodMoon"
        tuneBloodMoon(iAmt())
    end
    local function startSpecialAnim()
        if _specialConn then return end
        local accum = 0
        _specialConn = RunService.Heartbeat:Connect(function(dt)
            if not Config.Weather or not _special.kind then return end
            accum = accum + dt
            if accum < 0.1 then return end
            accum = 0
            local cam = Camera; if not cam then return end
            local pos = cam.CFrame.Position
            if _special.kind == "BloodMoon" and _special.moon then
                local cf = CFrame.new(pos + Vec(0.45, 0.62, -0.64).Unit * 700)
                _special.moon.CFrame = cf
                if _special.shell then _special.shell.CFrame = cf end
            end
        end)
    end
    local CSK, NSK = ColorSequenceKeypoint.new, NumberSequenceKeypoint.new
    local TX_MGLOW = "rbxassetid://78582616787441"
    local startMeteors, stopMeteors, rescheduleMeteors
    ;(function()
        local MET_TXS  = "rbxassetid://77935321198144"
        local MET_TXK  = "rbxassetid://102481842205398"
        local MET_TXD  = "rbxassetid://341828512"
        local MET_HAZE = Color3.fromRGB(128, 146, 176)
        local MET_GLOW = {
            { z = -0.80, size = 0.85, tr = 0.03, col = Color3.fromRGB(255, 246, 216), life = 0.09, rate = 34, b0 = 1.35, b1 = 2.1 },
            { z = -0.10, size = 1.75, tr = 0.55, col = Color3.fromRGB(255, 194, 106), life = 0.10, rate = 30, b0 = 0.85, b1 = 1.4 },
            { z =  1.30, size = 3.10, tr = 0.80, col = Color3.fromRGB(255, 124,  46), life = 0.11, rate = 26, b0 = 0.5, b1 = 0.8 },
            { z =  3.20, size = 5.20, tr = 0.92, col = Color3.fromRGB(224,  72,  26), life = 0.12, rate = 22, b0 = 0.4, b1 = 0.6 },
        }
        local MET_ROCK = {
            { 1.00, 0.74, 1.28,  0.00,  0.00,  0.00, 0.25, 0.40, 0.18, 58, 51, 47, 0.00, true },
            { 0.70, 0.60, 0.76,  0.40,  0.20,  0.12, 0.80, 0.50, 1.10, 47, 41, 39, 0.00, true },
            { 0.58, 0.48, 0.64, -0.38, -0.26,  0.34, 1.20, 0.90, 0.30, 36, 31, 30, 0.00, true },
            { 0.46, 0.40, 0.52,  0.06, -0.42, -0.24, 0.40, 1.30, 0.70, 52, 45, 42, 0.00, true },
            { 0.36, 0.30, 0.40, -0.32,  0.36, -0.08, 0.90, 0.20, 1.40, 42, 36, 34, 0.00, true },
            { 0.30, 0.26, 0.34,  0.22, -0.10,  0.46, 1.50, 0.70, 0.20, 38, 33, 32, 0.00, true },
            { 0.26, 0.34, 0.22, -0.20, -0.34, -0.02, 0.60, 1.10, 0.90, 33, 29, 28, 0.00, true },
            { 0.52, 0.40, 0.16,  0.10,  0.06, -0.56, 0.00, 0.00, 0.50, 255, 244, 214, 0.10, false },
            { 0.30, 0.46, 0.14, -0.30, -0.14, -0.44, 0.00, 0.00, -0.90, 255, 190, 104, 0.28, false },
            { 0.34, 0.20, 0.12,  0.16, -0.34, -0.40, 0.00, 0.00, 0.20, 255, 158,  66, 0.36, false },
            { 0.22, 0.16, 0.10, -0.10,  0.34, -0.38, 0.00, 0.00, 1.20, 255, 214, 150, 0.30, false },
        }
        local MET_ABL0 = 8
        local MET_ABLA = { 0.06, 0.20, 0.26, 0.22 }
        local MET_ABLK = { 0.20, 0.24, 0.26, 0.26 }
        local MET_UP, MET_DOWN = Vec(0, 300, 0), Vec(0, -800, 0)
        local _metConn = nil
        local _metNextFar, _metNextNear = 0, 0
        local _metFar, _metNear = 0, 0
        local _metLive = {}
        local _metRp = RaycastParams.new()
        _metRp.FilterType = Enum.RaycastFilterType.Exclude
        local function hz(c, k)
            return c:Lerp(MET_HAZE, k)
        end
        local function metPart(sx, sy, sz, cf)
            local p = Instance.new("Part")
            p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false
            p.CastShadow = false; p.Massless = true
            p.TopSurface = Enum.SurfaceType.Smooth; p.BottomSurface = Enum.SurfaceType.Smooth
            p.Size = Vec(sx, sy, sz); p.CFrame = cf
            p:SetAttribute("WX_Custom", true)
            p.Parent = getLightFolder()
            return p
        end
        local function metFilter()
            local ch = nil
            if lp then ch = lp.Character end
            if ch then
                _metRp.FilterDescendantsInstances = { getFolder(), ch }
            else
                _metRp.FilterDescendantsInstances = { getFolder() }
            end
        end
        local function metSolveLand(base, out, tang)
            local d = 90 + math.random() * 110
            local lat = (math.random() - 0.5) * 160
            for _ = 1, 2 do
                local c = base + out * d + tang * lat
                local h = Workspace:Raycast(c + MET_UP, MET_DOWN, _metRp)
                if h then
                    return h.Position + Vec(0, 1.5, 0)
                end
                d = d * 0.5
                lat = lat * 0.5
            end
            return nil
        end
        local function metGlow(host, size, transp, color, life, rate)
            local g = Instance.new("ParticleEmitter")
            g:SetAttribute("WX_Custom", true)
            g.Texture = TX_MGLOW; g.Color = ColorSequence.new(color)
            g.LightEmission = 1; g.LightInfluence = 0
            g.Rate = rate; g.Lifetime = NumberRange.new(life, life)
            g.Size = NumberSequence.new(size); g.Transparency = NumberSequence.new(transp)
            g.Speed = NumberRange.new(0, 0); g.LockedToPart = true
            g.Parent = host
            return g
        end
        local function metRibbon(part, sep, c0, c1, a0, w1, life, emit, inf)
            local aT = Instance.new("Attachment"); aT.Position = Vec(0, sep * 0.5, 0); aT.Parent = part
            local aB = Instance.new("Attachment"); aB.Position = Vec(0, -sep * 0.5, 0); aB.Parent = part
            local tr = Instance.new("Trail")
            tr:SetAttribute("WX_Custom", true)
            tr.Attachment0 = aT; tr.Attachment1 = aB; tr.FaceCamera = true
            tr.Texture = TX_MGLOW; tr.TextureMode = Enum.TextureMode.Stretch; tr.TextureLength = 1
            tr.Color = ColorSequence.new(c0, c1)
            tr.Transparency = NumberSequence.new({ NSK(0, a0), NSK(0.6, a0 + (1 - a0) * 0.45), NSK(1, 1) })
            tr.WidthScale = NumberSequence.new({ NSK(0, 1), NSK(1, w1) })
            tr.Lifetime = life; tr.LightEmission = emit; tr.LightInfluence = inf
            tr.MinLength = 0.05
            tr.Parent = part
            return { t = tr, a = aT, b = aB, sep = sep }
        end
        local function metSmoke(host, sb, lo, hi, rate, aBase, drift, spread, k)
            local e = Instance.new("ParticleEmitter")
            e:SetAttribute("WX_Custom", true)
            e.Texture = MET_TXS
            e.Color = ColorSequence.new({
                CSK(0,   hz(Color3.fromRGB(178, 164, 152), k * 0.5)),
                CSK(0.2, hz(Color3.fromRGB(146, 142, 138), k * 0.5)),
                CSK(1,   hz(Color3.fromRGB(84, 82, 80), k * 0.5)) })
            e.LightEmission = 0.02; e.LightInfluence = 0.6
            e.Rate = rate; e.Lifetime = NumberRange.new(lo, hi)
            e.Size = NumberSequence.new({ NSK(0, sb), NSK(0.35, sb * 1.9), NSK(1, sb * 3) })
            e.Transparency = NumberSequence.new({
                NSK(0, 0.94), NSK(0.07, aBase + 0.16 * k), NSK(0.62, aBase + 0.16), NSK(1, 1) })
            e.Speed = NumberRange.new(0, spread); e.SpreadAngle = Vector2.new(60, 60)
            e.Drag = 0.5
            e.RotSpeed = NumberRange.new(-8, 8); e.Rotation = NumberRange.new(0, 360)
            e.Acceleration = Vec(Wind.x * drift, 0.35, Wind.z * drift)
            e.Parent = host
            return e
        end
        local function bez(p0, p1, p2, a)
            return p0:Lerp(p1, a):Lerp(p1:Lerp(p2, a), a)
        end
        local function metImpact(pos, sc)
            if Camera then
                sc = math.min(sc, math.max(0.85, (pos - Camera.CFrame.Position).Magnitude / 44))
            end
            local host = metPart(4 * sc, 4 * sc, 4 * sc, CFrame.new(pos))
            host.Shape = Enum.PartType.Ball; host.Material = Enum.Material.Neon
            host.Color = Color3.fromRGB(255, 240, 202); host.Transparency = 0.05
            local fl = Instance.new("PointLight")
            fl.Color = Color3.fromRGB(255, 176, 90); fl.Range = 150; fl.Brightness = 9; fl.Parent = host
            local glow = metPart(2, 2, 2, CFrame.new(pos + Vec(0, 3, 0)))
            glow.Transparency = 1
            local gl = Instance.new("PointLight")
            gl.Color = Color3.fromRGB(255, 148, 68); gl.Range = 120 * sc; gl.Brightness = 4; gl.Parent = glow
            local ring = metPart(12 * sc, 2 * sc, 12 * sc, CFrame.new(pos))
            ring.Shape = Enum.PartType.Ball; ring.Material = Enum.Material.Neon
            ring.Color = Color3.fromRGB(255, 150, 60); ring.Transparency = 0.35
            local scorch = metPart(15 * sc, 0.6, 15 * sc, CFrame.new(pos - Vec(0, 0.4, 0)))
            scorch.Shape = Enum.PartType.Ball; scorch.Material = Enum.Material.Neon
            scorch.Color = Color3.fromRGB(255, 104, 24); scorch.Transparency = 0.2
            local att = Instance.new("Attachment"); att.Parent = host
            local datt = Instance.new("Attachment"); datt.Orientation = Vec(0, 0, -90); datt.Parent = host
            local flash = Instance.new("ParticleEmitter")
            flash:SetAttribute("WX_Custom", true)
            flash.Texture = TX_MGLOW; flash.Rate = 0
            flash.Color = ColorSequence.new({
                CSK(0,    Color3.fromRGB(255, 246, 214)),
                CSK(0.45, Color3.fromRGB(255, 186, 96)),
                CSK(1,    Color3.fromRGB(226, 96, 34)) })
            flash.LightEmission = 1; flash.LightInfluence = 0
            flash.Lifetime = NumberRange.new(0.30, 0.30)
            flash.Size = NumberSequence.new({ NSK(0, 14 * sc), NSK(0.35, 34 * sc), NSK(1, 44 * sc) })
            flash.Transparency = NumberSequence.new({ NSK(0, 0.05), NSK(0.45, 0.42), NSK(1, 1) })
            flash.Speed = NumberRange.new(0, 0); flash.Rotation = NumberRange.new(0, 360)
            flash.Parent = att
            flash:Emit(2)
            local dust = Instance.new("ParticleEmitter")
            dust:SetAttribute("WX_Custom", true)
            dust.Texture = MET_TXD; dust.Rate = 0
            dust.Color = ColorSequence.new(Color3.fromRGB(198, 178, 152), Color3.fromRGB(120, 112, 104))
            dust.LightEmission = 0.05; dust.LightInfluence = 0.5
            dust.Lifetime = NumberRange.new(5.5, 9)
            dust.Size = NumberSequence.new({ NSK(0, 6 * sc), NSK(1, 22 * sc) })
            dust.Transparency = NumberSequence.new({ NSK(0, 0.6), NSK(0.1, 0.42), NSK(0.6, 0.68), NSK(1, 1) })
            dust.Speed = NumberRange.new(22, 38); dust.SpreadAngle = Vector2.new(0, 180)
            dust.Drag = 1.9; dust.EmissionDirection = ND.Top
            dust.RotSpeed = NumberRange.new(-16, 16); dust.Rotation = NumberRange.new(0, 360)
            dust.Acceleration = Vec(Wind.x * 6, 0.4, Wind.z * 6)
            dust.Parent = datt
            dust:Emit(math.floor(24 * sc))
            local fount = Instance.new("ParticleEmitter")
            fount:SetAttribute("WX_Custom", true)
            fount.Texture = MET_TXK; fount.Rate = 0
            fount.Color = ColorSequence.new(Color3.fromRGB(255, 200, 110), Color3.fromRGB(196, 52, 14))
            fount.LightEmission = 1; fount.LightInfluence = 0
            fount.Lifetime = NumberRange.new(1.1, 2.2)
            fount.Size = NumberSequence.new({ NSK(0, 1.7 * sc), NSK(1, 0.08) })
            fount.Transparency = NumberSequence.new({ NSK(0, 0), NSK(0.7, 0.35), NSK(1, 1) })
            fount.Orientation = Enum.ParticleOrientation.VelocityParallel
            fount.Speed = NumberRange.new(48, 96); fount.SpreadAngle = Vector2.new(30, 30)
            fount.Acceleration = Vec(0, -74, 0); fount.Drag = 0.35
            fount.EmissionDirection = ND.Top; fount.Parent = att
            fount:Emit(math.floor(52 * sc))
            local col = Instance.new("ParticleEmitter")
            col:SetAttribute("WX_Custom", true)
            col.Texture = TX_MGLOW; col.Rate = 0
            col.Color = ColorSequence.new({
                CSK(0,    Color3.fromRGB(255, 236, 180)),
                CSK(0.35, Color3.fromRGB(255, 130, 40)),
                CSK(1,    Color3.fromRGB(122, 40, 18)) })
            col.LightEmission = 0.9; col.LightInfluence = 0
            col.Lifetime = NumberRange.new(0.55, 1.05)
            col.Size = NumberSequence.new({ NSK(0, 5 * sc), NSK(1, 14 * sc) })
            col.Transparency = NumberSequence.new({ NSK(0, 0.12), NSK(0.6, 0.55), NSK(1, 1) })
            col.Speed = NumberRange.new(32, 62); col.SpreadAngle = Vector2.new(10, 10)
            col.EmissionDirection = ND.Top; col.Parent = att
            col:Emit(18)
            local pil = Instance.new("ParticleEmitter")
            pil:SetAttribute("WX_Custom", true)
            pil.Texture = MET_TXS; pil.Rate = 0
            pil.Color = ColorSequence.new({
                CSK(0,   Color3.fromRGB(186, 150, 118)),
                CSK(0.3, Color3.fromRGB(146, 140, 132)),
                CSK(1,   Color3.fromRGB(88, 85, 82)) })
            pil.LightEmission = 0.02; pil.LightInfluence = 0.5
            pil.Lifetime = NumberRange.new(4.5, 8)
            pil.Size = NumberSequence.new({ NSK(0, 4.5 * sc), NSK(1, 20 * sc) })
            pil.Transparency = NumberSequence.new({ NSK(0, 0.52), NSK(0.5, 0.68), NSK(1, 1) })
            pil.Speed = NumberRange.new(16, 30); pil.SpreadAngle = Vector2.new(9, 9)
            pil.Acceleration = Vec(Wind.x * 7, 2.6, Wind.z * 7); pil.Drag = 0.6
            pil.EmissionDirection = ND.Top
            pil.RotSpeed = NumberRange.new(-12, 12); pil.Rotation = NumberRange.new(0, 360)
            pil.Parent = att
            task.delay(0.12, function() if pil.Parent then pil:Emit(7) end end)
            task.delay(0.62, function() if pil.Parent then pil:Emit(5) end end)
            task.delay(1.30, function() if pil.Parent then pil:Emit(4) end end)
            local t0 = tick()
            local conn
            conn = RunService.Heartbeat:Connect(function()
                if not host.Parent then
                    conn:Disconnect()
                    return
                end
                local k = tick() - t0
                local f1 = math.clamp(k / 0.10, 0, 1)
                host.Size = Vec(1, 1, 1) * (4 + 6 * f1) * sc
                host.Transparency = 0.05 + 0.95 * f1
                fl.Brightness = 9 * (1 - math.clamp(k / 0.55, 0, 1))
                gl.Brightness = 4 * (1 - math.clamp(k / 5, 0, 1))
                local f2 = math.clamp(k / 0.55, 0, 1)
                ring.Size = Vec(12 + 52 * f2, 2 - 1.2 * f2, 12 + 52 * f2) * sc
                ring.Transparency = 0.35 + 0.65 * f2
                scorch.Transparency = 0.2 + 0.8 * math.clamp(k / 7, 0, 1)
                if k > 7.2 then
                    conn:Disconnect()
                end
            end)
            Debris:AddItem(ring, 1.1)
            Debris:AddItem(glow, 5.2)
            Debris:AddItem(scorch, 7.5)
            Debris:AddItem(host, 11)
        end
        local function launch(far, opts)
            local ang = math.random() * 6.28318
            local mul = 1
            local big = false
            if opts then
                if opts.ang then ang = opts.ang end
                if opts.scaleMul then mul = opts.scaleMul end
                big = opts.big == true
            end
            local tang = Vec(-math.sin(ang), 0, math.cos(ang))
            local out = Vec(math.cos(ang), 0, math.sin(ang))
            local sgn = 1
            if math.random() < 0.5 then sgn = -1 end
            local base = Camera.CFrame.Position
            local startP, endP, spd, rs, ws, k, doImpact, smk
            if far then
                local ctr = base + out * (1400 + math.random() * 1200)
                          + Vec(0, 820 + math.random() * 360, 0)
                local travel = 3000 + math.random() * 900
                local drop = 320 + math.random() * 200
                startP = ctr - tang * (travel * 0.5 * sgn) + Vec(0, drop * 0.5, 0)
                endP = ctr + tang * (travel * 0.5 * sgn) - Vec(0, drop * 0.5, 0)
                spd = 255 + math.random() * 70
                rs = 14 + math.random() * 5
                ws = 3.9 + math.random()
                k = 0.14 + math.random() * 0.10
                doImpact = false
                smk = 1.6
            else
                metFilter()
                local land = nil
                if opts and opts.land then
                    local h = Workspace:Raycast(opts.land + MET_UP, MET_DOWN, _metRp)
                    if h then land = h.Position + Vec(0, 1.5, 0) end
                end
                if not land then land = metSolveLand(base, out, tang) end
                doImpact = land ~= nil
                if not land then
                    land = base + out * 150 + Vec(0, 40, 0)
                end
                local travel = 1750 + math.random() * 450
                startP = land - tang * (travel * sgn) + out * (190 + math.random() * 190)
                         + Vec(0, 640 + math.random() * 160, 0)
                if opts and opts.start then startP = opts.start end
                endP = land
                spd = 175 + math.random() * 38
                rs = (7.0 + math.random() * 2.6) * mul
                ws = (1.3 + math.random() * 0.28) * mul
                k = 0
                smk = 1
            end
            if big then
                rs = rs * 1.7
                ws = ws * 1.5
                spd = spd * 0.92
            end
            local flare = far or (not doImpact)
            local splitAt = nil
            if big and not (opts and opts.noSplit) then
                splitAt = 0.42 + math.random() * 0.12
            end
            local mid = startP:Lerp(endP, 0.5) + Vec(0, (endP - startP).Magnitude * 0.075, 0)
            local dur = ((startP - mid).Magnitude + (mid - endP).Magnitude) / spd
            local cf0 = CFrame.lookAt(startP, endP)
            local root = metPart(0.2, 0.2, 0.2, cf0)
            root.Transparency = 1
            local a0 = Instance.new("Attachment"); a0.Parent = root
            local glows = table.create(4)
            for i, s in MET_GLOW do
                local hp = metPart(0.2, 0.2, 0.2, cf0)
                hp.Transparency = 1
                glows[i] = { p = hp, z = s.z * rs, b0 = s.b0, b1 = s.b1,
                    e = metGlow(hp, s.size * rs, math.min(0.97, s.tr + 0.18 * k),
                        hz(s.col, k * 0.55), s.life, s.rate) }
            end
            local chunks = table.create(#MET_ROCK)
            for i, r in MET_ROCK do
                local p = metPart(r[1] * rs, r[2] * rs, r[3] * rs, cf0)
                p.Transparency = r[13]
                if r[14] then
                    p.Material = Enum.Material.Slate
                    p.Color = Color3.fromRGB(r[10], r[11], r[12])
                else
                    p.Material = Enum.Material.Neon
                    p.Color = hz(Color3.fromRGB(r[10], r[11], r[12]), k * 0.4)
                end
                chunks[i] = { p = p, tumble = r[14],
                    off = CFrame.new(r[4] * rs, r[5] * rs, r[6] * rs) * CFrame.Angles(r[7], r[8], r[9]) }
            end
            local noseLight, bodyLight = nil, nil
            if not far then
                noseLight = Instance.new("PointLight")
                noseLight.Color = Color3.fromRGB(255, 186, 108)
                noseLight.Range = 4.2 * rs; noseLight.Brightness = 11
                noseLight.Parent = glows[1].p
                bodyLight = Instance.new("PointLight")
                bodyLight.Color = Color3.fromRGB(255, 150, 70)
                bodyLight.Range = 16 * rs; bodyLight.Brightness = 3
                bodyLight.Parent = root
            end
            local ha = 0.20 * k
            local ribbons = {
                metRibbon(root, 7.5 * ws, hz(Color3.fromRGB(255, 253, 248), k * 0.3),
                    hz(Color3.fromRGB(255, 226, 156), k * 0.6), ha, 0.45, 0.26, 1, 0),
                metRibbon(root, 14 * ws, hz(Color3.fromRGB(255, 222, 146), k * 0.5),
                    hz(Color3.fromRGB(255, 136, 40), k * 0.9), 0.06 + ha, 0.36, 0.72, 1, 0),
                metRibbon(root, 23 * ws, hz(Color3.fromRGB(255, 138, 42), k * 0.9),
                    hz(Color3.fromRGB(196, 50, 14), k), 0.22 + ha, 0.28, 1.85, 1, 0),
                metRibbon(root, 33 * ws, hz(Color3.fromRGB(196, 62, 18), k),
                    hz(Color3.fromRGB(88, 26, 14), k), 0.52 + ha, 0.30, 3.20, 0.55, 0.12),
            }
            local trainRate = math.clamp(spd / (17 * ws * 0.30 * smk), 6, 30)
            local train = metSmoke(a0, 17 * ws, 5, 7.5, trainRate, 0.56, 5.6, 3, k)
            local scarRate = math.clamp(spd / (50 * ws * 0.28 * smk), 1.6, 9)
            local scar = metSmoke(a0, 50 * ws, 20, 32, scarRate, 0.74, 7.8, 7, k)
            local sparks = nil
            if not far then
                sparks = Instance.new("ParticleEmitter")
                sparks:SetAttribute("WX_Custom", true)
                sparks.Texture = MET_TXK
                sparks.Color = ColorSequence.new(Color3.fromRGB(255, 172, 70), Color3.fromRGB(186, 44, 12))
                sparks.LightEmission = 1; sparks.LightInfluence = 0
                sparks.Rate = 60; sparks.Lifetime = NumberRange.new(0.6, 1.5)
                sparks.Size = NumberSequence.new({ NSK(0, 2.2 * ws), NSK(1, 0.06) })
                sparks.Transparency = NumberSequence.new({ NSK(0, 0.08), NSK(1, 1) })
                sparks.Orientation = Enum.ParticleOrientation.VelocityParallel
                sparks.Speed = NumberRange.new(6, 20); sparks.SpreadAngle = Vector2.new(32, 32)
                sparks.Acceleration = Vec(0, -12, 0); sparks.Drag = 0.6
                sparks.Parent = a0
            end
            if far then _metFar = _metFar + 1 else _metNear = _metNear + 1 end
            local rec = { root = root }
            _metLive[#_metLive + 1] = rec
            local t0 = tick()
            local ph = math.random() * 20
            local spinX, spinY, spinZ = (math.random() - 0.5) * 3, (math.random() - 0.5) * 3.4,
                0.6 + math.random() * 1.6
            local flareAt = 0.76 + math.random() * 0.10
            local shedAt = 0.25 + math.random() * 0.2
            local dead, endT = false, nil
            local conn
            local function retire()
                if dead then
                    return
                end
                dead = true
                if far then
                    _metFar = math.max(0, _metFar - 1)
                else
                    _metNear = math.max(0, _metNear - 1)
                end
                for i = #_metLive, 1, -1 do
                    if _metLive[i] == rec then table.remove(_metLive, i) end
                end
            end
            conn = RunService.Heartbeat:Connect(function()
                if not root.Parent then
                    conn:Disconnect()
                    retire()
                    return
                end
                local now = tick()
                local a = math.clamp((now - t0) / dur, 0, 1)
                local p = bez(startP, mid, endP, a)
                local d = (mid - startP) * (2 * (1 - a)) + (endP - mid) * (2 * a)
                if d.Magnitude < 1e-3 then d = endP - startP end
                local cf = CFrame.lookAt(p, p + d)
                local n = math.clamp(0.5 + 0.27 * math.sin(now * 41 + ph)
                    + 0.17 * math.sin(now * 67.3 + ph * 2.1)
                    + 0.09 * math.sin(now * 103.7 + ph * 0.7), 0, 1)
                local burn = 1
                if flare and a > flareAt then
                    local u = (a - flareAt) / (1 - flareAt)
                    burn = math.clamp((1 + 1.7 * math.exp(-((u - 0.14) ^ 2) / 0.011))
                        * (1 - u) ^ 1.5, 0, 3)
                end
                if endT then burn = burn * math.clamp(1 - (now - endT) / 0.4, 0, 1) end
                local bw = math.clamp(burn, 0, 1)
                if not endT then
                    root.CFrame = cf
                    for _, g in glows do
                        g.p.CFrame = cf * CFrame.new(0, 0, g.z)
                    end
                    local tum = CFrame.Angles(now * spinX, now * spinY, now * spinZ)
                    for _, c in chunks do
                        if c.tumble then
                            c.p.CFrame = cf * tum * c.off
                        else
                            c.p.CFrame = cf * c.off
                        end
                    end
                end
                for _, g in glows do
                    g.e.Brightness = (g.b0 + g.b1 * n) * burn
                end
                for i = 1, 4 do
                    local c = chunks[MET_ABL0 + i - 1]
                    c.p.Transparency = math.clamp(1 - (1 - (MET_ABLA[i] + MET_ABLK[i] * (1 - n))) * bw, 0, 1)
                end
                if noseLight then
                    noseLight.Brightness = (7 + 9 * n) * bw
                    bodyLight.Brightness = (2 + 3 * n) * bw
                end
                train.Rate = trainRate * bw
                scar.Rate = scarRate * bw
                if sparks then sparks.Rate = 60 * bw * (0.6 + 0.8 * n) end
                local pinch = (0.5 + 0.5 * bw) * (0.9 + 0.2 * n)
                for _, L in ribbons do
                    local h = L.sep * 0.5 * pinch
                    L.a.Position = Vec(0, h, 0)
                    L.b.Position = Vec(0, -h, 0)
                end
                if splitAt and a >= splitAt then
                    splitAt = nil
                    if sparks then sparks:Emit(30) end
                    for _ = 1, math.random(2, 3) do
                        launch(false, {
                            ang = ang,
                            noSplit = true,
                            scaleMul = 0.40 + math.random() * 0.14,
                            start = p + Vec((math.random() - 0.5) * 30, (math.random() - 0.5) * 20,
                                (math.random() - 0.5) * 30),
                            land = endP + Vec((math.random() - 0.5) * 110, 0, (math.random() - 0.5) * 110),
                        })
                    end
                end
                if (not far) and a > shedAt then
                    shedAt = 2
                    local fp = metPart(0.3 * rs, 0.24 * rs, 0.38 * rs, CFrame.new(p))
                    fp.Material = Enum.Material.Slate
                    fp.Color = Color3.fromRGB(40, 35, 33)
                    metRibbon(fp, 6, Color3.fromRGB(255, 220, 160), Color3.fromRGB(150, 34, 12),
                        0.12, 0.2, 0.8, 1, 0)
                    local fdir = (d.Unit * 0.65
                        + Vec(math.random() - 0.5, -0.5, math.random() - 0.5).Unit * 0.5).Unit
                    local fv, ft0 = fdir * spd * 0.6, tick()
                    local fc
                    fc = RunService.Heartbeat:Connect(function()
                        if not fp.Parent then
                            fc:Disconnect()
                            return
                        end
                        local age = tick() - ft0
                        if age > 2.4 then
                            fc:Disconnect()
                            fp:Destroy()
                            return
                        end
                        fv = fv + Vec(0, -0.88, 0)
                        fp.CFrame = CFrame.new(fp.Position + fv * 0.016)
                            * CFrame.Angles(age * 5, age * 3, age * 4)
                    end)
                end
                if not endT then
                    if a >= 1 then
                        endT = now
                        if doImpact then
                            metImpact(endP, rs * 0.215 + 0.6)
                            local tid = cfg("WeatherThunderId", "")
                            if tid ~= "" then
                                task.delay(0.15, function()
                                    oneShot(tid, cfg("WeatherSoundVolume", 0.35) * 6)
                                end)
                            end
                        end
                    elseif flare and burn < 0.02 and a > flareAt then
                        endT = now
                    end
                elseif now - endT >= 0.4 then
                    retire()
                    conn:Disconnect()
                    train.Enabled = false; scar.Enabled = false
                    if sparks then sparks.Enabled = false end
                    for _, g in glows do
                        g.e.Enabled = false
                        g.p:Destroy()
                    end
                    for _, L in ribbons do L.t.Enabled = false end
                    for _, c in chunks do c.p:Destroy() end
                    if noseLight then noseLight:Destroy(); bodyLight:Destroy() end
                    Debris:AddItem(root, 42)
                end
            end)
            Debris:AddItem(root, dur + 46)
        end
        local function spawnMeteor(far, opts)
            if not Config.Weather or not cfg("WeatherMeteors", false) then
                return
            end
            local r = evRate("WeatherMeteorRate")
            local capped
            if far then
                capped = _metFar >= math.clamp(math.floor(1 + 1.5 * r), 1, 4)
            elseif r >= 2.5 then
                capped = _metNear >= 3
            elseif r >= 1.5 then
                capped = _metNear >= 2
            else
                capped = _metNear >= 1
            end
            if capped then
                return
            end
            launch(far, opts)
        end
        startMeteors = function()
            if _metConn then
                return
            end
            local t = tick()
            _metNextFar = t + 1 + math.random() * 2
            _metNextNear = t + 5 + math.random() * 6
            _metConn = RunService.Heartbeat:Connect(function()
                if not Config.Weather or not cfg("WeatherMeteors", false) then
                    return
                end
                local now = tick()
                local r = evRate("WeatherMeteorRate")
                if now >= _metNextFar then
                    _metNextFar = now + (6 + math.random() * 5) / r
                    pcall(spawnMeteor, true)
                end
                if now >= _metNextNear then
                    _metNextNear = now + (14 + math.random() * 10) / r
                    pcall(spawnMeteor, false)
                end
            end)
        end
        rescheduleMeteors = function()
            if not _metConn then
                return
            end
            local now, r = tick(), evRate("WeatherMeteorRate")
            _metNextFar = math.min(_metNextFar, now + 11 / r)
            _metNextNear = math.min(_metNextNear, now + 24 / r)
        end
        stopMeteors = function()
            if _metConn then _metConn:Disconnect(); _metConn = nil end
            for _, rec in _metLive do
                pcall(function() rec.root:Destroy() end)
            end
            table.clear(_metLive)
        end
        registerSpectacle("MeteorBigOne", 1, function()
            if not Config.Weather or not cfg("WeatherMeteors", false) then
                return
            end
            launch(false, { big = true })
        end)
    end)()
    local startStars, stopStars, rescheduleStars
    ;(function()
        local TX_SGLOW = "rbxassetid://78582616787441"
        local TX_STAR4 = "rbxassetid://17726943419"
        local GLINT_C  = Color3.fromRGB(246, 250, 255)
        local MIN_SIN  = 0.3
        local FLOOR_H  = 60
        local PAL = {
            { WHITE, Color3.fromRGB(222, 234, 255), Color3.fromRGB(150, 184, 240) },
            { Color3.fromRGB(242, 250, 255), Color3.fromRGB(160, 206, 255), Color3.fromRGB(78, 138, 255) },
            { Color3.fromRGB(255, 246, 220), Color3.fromRGB(255, 205, 122), Color3.fromRGB(222, 140, 44) },
        }
        local CLS = {
            { 80, 55, 250, 105, 50, 28, 3.6 },
            { 145, 90, 158, 62, 76, 44, 5.4 },
            { 240, 150, 98, 46, 118, 62, 7.2 },
        }
        local TR_TRANSP  = NumberSequence.new({ NSK(0, 0.04), NSK(0.12, 0.12), NSK(0.45, 0.55), NSK(1, 1) })
        local TR_WIDTH   = NumberSequence.new({ NSK(0, 0.4), NSK(0.08, 1), NSK(1, 0.02) })
        local ION_TRANSP = NumberSequence.new({ NSK(0, 0.86), NSK(0.3, 0.9), NSK(1, 1) })
        local ION_WIDTH  = NumberSequence.new({ NSK(0, 0.5), NSK(0.25, 1), NSK(1, 0.35) })
        local HEAD_TR    = NumberSequence.new({ NSK(0, 1), NSK(0.05, 0), NSK(0.72, 0.06), NSK(1, 1) })
        local HALO_TR    = NumberSequence.new({ NSK(0, 1), NSK(0.07, 0.82), NSK(0.7, 0.88), NSK(1, 1) })
        local GLINT_TR   = NumberSequence.new({ NSK(0, 1), NSK(0.3, 0.12), NSK(0.55, 0.42), NSK(1, 1) })
        local _starConn, _starNextT = nil, 0
        local _starLive = {}
        local function mkSprite(parent, tex, col, sizeSeq, transpSeq, life, emit)
            local e = Instance.new("ParticleEmitter")
            e.Texture = tex
            e.Color = ColorSequence.new(col)
            e.Size = sizeSeq
            e.Transparency = transpSeq
            e.Lifetime = NumberRange.new(life)
            e.Rate = 0
            e.Speed = NumberRange.new(0, 0)
            e.SpreadAngle = Vector2.new(0, 0)
            e.LightEmission = emit
            e.LightInfluence = 0
            e.Drag = 0
            e.Parent = parent
            return e
        end
        local function spawnStreak(start, dir, ci, pi, floorY)
            if not Config.Weather or not cfg("WeatherShootingStars", false) then
                return
            end
            if #_starLive >= math.clamp(math.floor(4 + 2 * evRate("WeatherStarRate")), 4, 8) then
                return
            end
            local cls, pal = CLS[ci], PAL[pi]
            local dist = cls[1] + math.random() * cls[2]
            local spd  = cls[3] + math.random() * cls[4]
            local tail = cls[5] + math.random() * cls[6]
            if start.Y + dir.Y * dist < floorY then
                local ny = (floorY - start.Y) / dist
                local hl = math.sqrt(dir.X * dir.X + dir.Z * dir.Z)
                if hl > 1e-4 then
                    local k = math.sqrt(math.max(0, 1 - ny * ny)) / hl
                    dir = Vec(dir.X * k, ny, dir.Z * k)
                end
            end
            local dur  = dist / spd
            local life = math.min(tail / spd, dur * 0.9)
            local sep  = cls[7] * (0.85 + math.random() * 0.3)
            local glintD = 0.3 + math.random() * 0.3
            local cf0 = CFrame.lookAt(start, start + dir)
            local host = mkBoltPart(WHITE, 1, Vec(0.2, 0.2, 0.2), cf0, getLightFolder())
            local aT = Instance.new("Attachment"); aT.Position = Vec(0, sep * 0.5, 0); aT.Parent = host
            local aB = Instance.new("Attachment"); aB.Position = Vec(0, -sep * 0.5, 0); aB.Parent = host
            local tr = Instance.new("Trail")
            tr.Attachment0 = aT; tr.Attachment1 = aB
            tr.FaceCamera = true
            tr.Texture = TX_SGLOW; tr.TextureMode = Enum.TextureMode.Stretch; tr.TextureLength = 1
            tr.Color = ColorSequence.new({ CSK(0, pal[1]), CSK(0.24, pal[2]), CSK(1, pal[3]) })
            tr.Transparency = TR_TRANSP; tr.WidthScale = TR_WIDTH
            tr.Lifetime = life; tr.LightEmission = 1; tr.LightInfluence = 0
            tr.MinLength = 0.08; tr.Enabled = false
            tr.Parent = host
            local ion = nil
            if ci == 3 then
                ion = Instance.new("Trail")
                ion.Attachment0 = aT; ion.Attachment1 = aB
                ion.FaceCamera = true
                ion.Texture = TX_SGLOW; ion.TextureMode = Enum.TextureMode.Stretch; ion.TextureLength = 1
                ion.Color = ColorSequence.new({ CSK(0, pal[2]), CSK(1, pal[3]) })
                ion.Transparency = ION_TRANSP; ion.WidthScale = ION_WIDTH
                ion.Lifetime = life * 2.6; ion.LightEmission = 0.85; ion.LightInfluence = 0
                ion.MinLength = 0.1; ion.Enabled = false
                ion.Parent = host
            end
            local hub = Instance.new("Attachment"); hub.Parent = host
            local coreS = sep * 0.62
            local core = mkSprite(hub, TX_SGLOW, pal[1], NumberSequence.new({
                NSK(0, coreS * 0.55), NSK(0.1, coreS), NSK(1, coreS * 0.3) }), HEAD_TR, dur, 1)
            core.LockedToPart = true
            local haloS = sep * 1.7
            local halo = mkSprite(hub, TX_SGLOW, pal[2], NumberSequence.new({
                NSK(0, haloS * 0.5), NSK(0.12, haloS), NSK(1, haloS * 0.35) }), HALO_TR, dur, 1)
            halo.LockedToPart = true
            local gs = sep * 1.9
            local gl = mkSprite(hub, TX_STAR4, GLINT_C, NumberSequence.new({
                NSK(0, gs * 0.1), NSK(0.32, gs), NSK(1, gs * 0.14) }), GLINT_TR, glintD * 1.2, 0.55)
            gl.Rotation = NumberRange.new(0, 90)
            gl.RotSpeed = NumberRange.new(-16, 16)
            gl:Emit(1)
            table.insert(_starLive, {
                host = host, tr = tr, ion = ion, core = core, halo = halo, aT = aT, aB = aB,
                cf0 = cf0, dir = dir, dist = dist, dur = dur, sep = sep,
                t0 = tick() + glintD, started = false,
            })
            Debris:AddItem(host, glintD + dur + life * 2.8 + 0.6)
        end
        local function pickPal()
            local r = math.random()
            if r < 0.55 then
                return 1
            end
            if r < 0.92 then
                return 2
            end
            return 3
        end
        local function pickCls(pi)
            if pi == 3 then
                if math.random() < 0.6 then
                    return 3
                end
                return 2
            end
            local r = math.random()
            if r < 0.25 then
                return 1
            end
            if r < 0.7 then
                return 2
            end
            return 3
        end
        local function viewAz(cam)
            local lv = cam.CFrame.LookVector
            local az = math.atan2(lv.Z, lv.X)
            if math.random() < 0.35 then
                return math.random() * 6.283
            end
            return az
        end
        local function spawnSingle()
            local cam = Camera
            if not cam then
                return
            end
            local base = cam.CFrame.Position
            local az = viewAz(cam) + (math.random() - 0.5) * 1.5
            local el = math.rad(24 + math.random() * 30)
            local r  = 190 + math.random() * 130
            local ce = math.cos(el)
            local start = base + Vec(ce * math.cos(az) * r, math.sin(el) * r, ce * math.sin(az) * r)
            local hd = az + 1.5708 + (math.random() - 0.5) * 2.2
            local pi = pickPal()
            spawnStreak(start, Vec(math.cos(hd), -(0.05 + math.random() * 0.34), math.sin(hd)).Unit,
                pickCls(pi), pi, base.Y + FLOOR_H)
        end
        local function spawnShower()
            local cam = Camera
            if not cam then
                return
            end
            local base = cam.CFrame.Position
            local floorY = base.Y + FLOOR_H
            local raz = viewAz(cam) + (math.random() - 0.5) * 0.9
            local rel = math.rad(36 + math.random() * 16)
            local cr = math.cos(rel)
            local rv = Vec(cr * math.cos(raz), math.sin(rel), cr * math.sin(raz))
            local dn = (Vec(0, -1, 0) + rv * rv.Y).Unit
            local sd = rv:Cross(dn).Unit
            local pi = pickPal()
            local t = 0
            for _ = 1, 3 + math.random(0, 2) do
                t = t + 0.16 + math.random() * 0.42
                task.delay(t, function()
                    if not Config.Weather or not cfg("WeatherShootingStars", false) then
                        return
                    end
                    local phi = (math.random() - 0.5) * 2.8
                    local off = dn * math.cos(phi) + sd * math.sin(phi)
                    local th = math.rad(9 + math.random() * 24)
                    local ct, st = math.cos(th), math.sin(th)
                    local u = (rv * ct + off * st).Unit
                    if u.Y < MIN_SIN then
                        off = -off
                        u = (rv * ct + off * st).Unit
                    end
                    if u.Y < MIN_SIN then
                        return
                    end
                    local ci = 2
                    if th < 0.22 then
                        ci = 1
                    elseif th > 0.44 then
                        ci = 3
                    end
                    pcall(spawnStreak, base + u * (215 + math.random() * 75),
                        (off * ct - rv * st).Unit, ci, pi, floorY)
                end)
            end
        end
        local function stepStars(now)
            for i = #_starLive, 1, -1 do
                local s = _starLive[i]
                if not s.host.Parent then
                    table.remove(_starLive, i)
                else
                    local a = (now - s.t0) / s.dur
                    if a >= 1 then
                        s.tr.Enabled = false
                        if s.ion then
                            s.ion.Enabled = false
                        end
                        table.remove(_starLive, i)
                    elseif a >= 0 then
                        if not s.started then
                            s.started = true
                            s.tr.Enabled = true
                            s.core:Emit(1)
                            s.halo:Emit(1)
                            if s.ion then
                                s.ion.Enabled = true
                            end
                        end
                        s.host.CFrame = s.cf0 + s.dir * (s.dist * a)
                        if a > 0.7 then
                            local h = s.sep * 0.5 * (1 - (a - 0.7) / 0.3)
                            s.aT.Position = Vec(0, h, 0)
                            s.aB.Position = Vec(0, -h, 0)
                        end
                    end
                end
            end
        end
        startStars = function()
            if _starConn then
                return
            end
            _starNextT = tick() + 2 + math.random() * 4
            _starConn = RunService.Heartbeat:Connect(function()
                if not Config.Weather or not cfg("WeatherShootingStars", false) then
                    return
                end
                local now = tick()
                if now >= _starNextT then
                    local r = evRate("WeatherStarRate")
                    _starNextT = now + (5 + math.random() * 6) / r
                    if math.random() < math.min(0.06 * r, 0.35) then
                        pcall(spawnShower)
                    else
                        pcall(spawnSingle)
                        if math.random() < math.min(0.22 * r, 0.6) then
                            task.delay(0.3 + math.random() * 0.5, function()
                                pcall(spawnSingle)
                            end)
                        end
                    end
                end
                pcall(stepStars, now)
            end)
        end
        rescheduleStars = function()
            if not _starConn then
                return
            end
            _starNextT = math.min(_starNextT, tick() + 11 / evRate("WeatherStarRate"))
        end
        stopStars = function()
            if _starConn then
                _starConn:Disconnect()
                _starConn = nil
            end
            for _, s in _starLive do
                pcall(function() s.host:Destroy() end)
            end
            table.clear(_starLive)
        end
        registerSpectacle("StarShower", 1, function()
            if not cfg("WeatherShootingStars", false) then
                return
            end
            spawnShower()
        end)
    end)()
    local startFireflies, stopFireflies, refreshFireflies
    ;(function()
        local FF_N     = 7
        local FF_BODY  = Color3.fromRGB(215, 255, 120)
        local FF_LIGHT = Color3.fromRGB(244, 232, 120)
        local FF_DOWN  = Vec(0, -80, 0)
        local WANDER   = 28
        local FF_STEP  = 2
        local FF_SEG   = 1.2
        local FF_TRC = ColorSequence.new({ CSK(0, Color3.fromRGB(198, 255, 128)),
            CSK(0.45, Color3.fromRGB(216, 238, 108)), CSK(1, Color3.fromRGB(255, 196, 86)) })
        local FF_TRT = NumberSequence.new({ NSK(0, 0.36), NSK(0.35, 0.66), NSK(1, 1) })
        local FF_TRW = NumberSequence.new({ NSK(0, 0.8), NSK(0.3, 1), NSK(1, 0) })
        local _ff = { list = nil, conn = nil, folder = nil, burst = nil, burstHost = nil, n = 4 }
        local _swarm = { active = false, t0 = 0, cx = 0, cz = 0, y = 0 }
        local _ffRp = RaycastParams.new()
        _ffRp.FilterType = Enum.RaycastFilterType.Exclude
        local function groundY(camPos, x, z)
            local ch = lp and lp.Character
            if ch then
                _ffRp.FilterDescendantsInstances = { getFolder(), ch }
            else
                _ffRp.FilterDescendantsInstances = { getFolder() }
            end
            local hit = Workspace:Raycast(Vec(x, camPos.Y + 6, z), FF_DOWN, _ffRp)
            if hit then
                return hit.Position.Y
            end
            return camPos.Y - 7
        end
        local function pickWaypoint(H, camPos)
            local x = camPos.X + (math.random() - 0.5) * WANDER
            local z = camPos.Z + (math.random() - 0.5) * WANDER
            H.a = H.pos
            H.bpt = Vec(x, groundY(camPos, x, z) + 0.5 + math.random() * 3, z)
            H.dur = math.clamp((H.bpt - H.a).Magnitude / 2.2, 1.5, 6)
            H.t = 0
        end
        local function stepToward(cur, target, maxStep)
            local d = target - cur
            local m = d.Magnitude
            if m <= maxStep or m < 1e-4 then
                return target
            end
            return cur + d * (maxStep / m)
        end
        local function trailOff(H)
            H.calm = 0
            if H.trail.Enabled then
                H.trail.Enabled = false
                H.trail:Clear()
            end
        end
        local function buildFlies()
            local folder = Instance.new("Folder")
            folder.Name = "_wxFlies"; folder:SetAttribute("WX_Custom", true)
            folder.Parent = getFolder()
            _ff.folder = folder
            _ff.list = {}
            local cam = Camera
            local camPos
            if cam then camPos = cam.CFrame.Position else camPos = Vec(0, 0, 0) end
            for i = 1, FF_N do
                local b = mkBoltPart(FF_BODY, 0, Vec(0.25, 0.25, 0.25), CFrame.new(camPos), folder)
                b.Shape = Enum.PartType.Ball
                local light = Instance.new("PointLight")
                light.Color = FF_LIGHT; light.Range = 8; light.Brightness = 2.5
                light.Parent = b
                local halo = Instance.new("ParticleEmitter")
                halo:SetAttribute("WX_Custom", true)
                halo.Texture = TX_MGLOW
                halo.Color = ColorSequence.new(FF_BODY)
                halo.LightEmission = 1; halo.LightInfluence = 0
                halo.Size = NumberSequence.new(1.1, 1.5)
                halo.Transparency = NumberSequence.new({ NSK(0, 0.75), NSK(1, 1) })
                halo.Rate = 12; halo.Lifetime = NumberRange.new(0.25, 0.4)
                halo.Speed = NumberRange.new(0, 0)
                halo.LockedToPart = true
                halo.Parent = b
                local a0 = Instance.new("Attachment")
                a0.Position = Vec(0, 0.6, 0); a0.Parent = b
                local a1 = Instance.new("Attachment")
                a1.Position = Vec(0, -0.6, 0); a1.Parent = b
                local tr = Instance.new("Trail")
                tr:SetAttribute("WX_Custom", true)
                tr.Attachment0 = a0; tr.Attachment1 = a1
                tr.Texture = TX_MGLOW; tr.TextureMode = Enum.TextureMode.Stretch
                tr.Color = FF_TRC; tr.Transparency = FF_TRT; tr.WidthScale = FF_TRW
                tr.Lifetime = 0.65
                tr.LightEmission = 1; tr.LightInfluence = 0
                tr.FaceCamera = true
                tr.MinLength = 0.25; tr.MaxLength = 14
                tr.Enabled = false
                tr.Parent = b
                local H = { part = b, light = light, trail = tr, halo = halo, bm = 1,
                            calm = 0, jump = true,
                            pos = camPos, a = camPos, bpt = camPos, t = 0, dur = 1,
                            ph = math.random() * 6.283, spin = 1.7 + i * 0.25,
                            blinkT = math.random() * 3, period = 2.4 + math.random() * 1.4 }
                pickWaypoint(H, camPos)
                H.pos = H.bpt
                pickWaypoint(H, camPos)
                b.CFrame = CFrame.new(H.pos)
                _ff.list[i] = H
            end
            local bh = mkBoltPart(FF_BODY, 1, Vec(1.5, 1.5, 1.5), CFrame.new(camPos), folder)
            local be = Instance.new("ParticleEmitter")
            be:SetAttribute("WX_Custom", true)
            be.Texture = TX_MGLOW
            be.Color = ColorSequence.new(Color3.fromRGB(220, 255, 130), Color3.fromRGB(255, 200, 80))
            be.LightEmission = 0.95; be.LightInfluence = 0.05
            be.Size = NumberSequence.new({ NSK(0, 0.1), NSK(0.08, 0.62, 0.15), NSK(0.45, 0.5, 0.1), NSK(1, 0.08) })
            be.Transparency = NumberSequence.new({ NSK(0, 1), NSK(0.08, 0.08), NSK(0.4, 0.45), NSK(0.75, 0.85), NSK(1, 1) })
            be.Rate = 0; be.Lifetime = NumberRange.new(1.6, 2.8)
            be.Speed = NumberRange.new(0.8, 2.2); be.SpreadAngle = Vector2.new(180, 180)
            be.Drag = 2.5; be.EmissionDirection = Enum.NormalId.Top
            be.Parent = bh
            _ff.burst = be; _ff.burstHost = bh
        end
        refreshFireflies = function()
            local list = _ff.list
            if not list then
                return
            end
            local I = iAmt()
            _ff.n = math.clamp(math.floor(1.5 + 3 * I), 1, FF_N)
            local rm, bm = 0.8 + 0.2 * I, 0.7 + 0.3 * I
            for i = 1, FF_N do
                local H = list[i]
                local on = i <= _ff.n
                H.bm = bm
                H.light.Range = 8 * rm
                H.halo.Enabled = on
                if not on then
                    H.light.Brightness = 0
                    H.part.Transparency = 1
                    H.jump = true
                    trailOff(H)
                end
            end
        end
        startFireflies = function()
            if _ff.conn then return end
            buildFlies()
            refreshFireflies()
            local accum = 0
            _ff.conn = RunService.Heartbeat:Connect(function(dt)
                if not Config.Weather then return end
                accum = accum + dt
                if accum < 0.05 then return end
                local step = accum; accum = 0
                local cam = Camera; if not cam then return end
                local camPos = cam.CFrame.Position
                local now = tick()
                local sw = -1
                if _swarm.active then
                    sw = now - _swarm.t0
                    if sw > 9 then _swarm.active = false; sw = -1 end
                end
                local list = _ff.list
                for i = 1, _ff.n do
                    local H = list[i]
                    local prev = H.pos
                    local newPos
                    if sw >= 0 then
                        local ang = H.ph + now * H.spin
                        local r
                        if sw < 2.5 then r = 10 - 7.4 * (sw / 2.5)
                        elseif sw < 6.5 then r = 2.6
                        else r = 2.6 + (sw - 6.5) * 6 end
                        local target = Vec(_swarm.cx + math.cos(ang) * r,
                            _swarm.y + 0.55 * math.sin(now * 1.3 + H.ph * 2),
                            _swarm.cz + math.sin(ang) * r)
                        newPos = stepToward(prev, prev:Lerp(target, math.min(step * 3, 1)), FF_STEP)
                        H.bpt = newPos; H.t = 1; H.dur = 1
                    else
                        H.t = H.t + step
                        local relX, relZ = prev.X - camPos.X, prev.Z - camPos.Z
                        if H.t >= H.dur or (relX * relX + relZ * relZ) > 3600 then
                            pickWaypoint(H, camPos)
                        end
                        local u = H.t / H.dur
                        u = u * u * (3 - 2 * u)
                        newPos = stepToward(prev, H.a:Lerp(H.bpt, u), FF_STEP)
                    end
                    H.pos = newPos
                    if H.jump or (sw >= 0 and (sw < 2.5 or sw >= 6.4))
                        or (newPos - prev).Magnitude > FF_SEG then
                        H.jump = false
                        trailOff(H)
                    else
                        H.calm = H.calm + step
                        if H.calm >= 0.25 and not H.trail.Enabled then
                            H.trail:Clear()
                            H.trail.Enabled = true
                        end
                    end
                    H.part.CFrame = CFrame.new(H.pos.X, H.pos.Y + 0.3 * math.sin(now * 1.7 + H.ph), H.pos.Z)
                    local pulse = math.exp(-((now + H.blinkT) % H.period) * 1.4)
                    H.light.Brightness = (0.5 + 2.6 * pulse) * H.bm
                    H.part.Transparency = 0.4 * (1 - pulse)
                end
            end)
        end
        stopFireflies = function()
            if _ff.conn then _ff.conn:Disconnect(); _ff.conn = nil end
            if _ff.folder then pcall(function() _ff.folder:Destroy() end); _ff.folder = nil end
            _ff.list = nil; _ff.burst = nil; _ff.burstHost = nil
            _swarm.active = false
        end
        registerSpectacle("FireflySwarm", 1, function()
            if not Config.Weather or _curType ~= "Fireflies" or not _ff.list then return end
            local cam = Camera; if not cam then return end
            local camPos = cam.CFrame.Position
            local ang = math.random() * 6.283
            local d = 8 + math.random() * 8
            local cx = camPos.X + math.cos(ang) * d
            local cz = camPos.Z + math.sin(ang) * d
            _swarm.cx = cx; _swarm.cz = cz
            _swarm.y = groundY(camPos, cx, cz) + 2.2
            _swarm.t0 = tick(); _swarm.active = true
            _ff.burstHost.CFrame = CFrame.new(cx, _swarm.y, cz)
            for _, H in _ff.list do
                trailOff(H)
            end
            local em = math.max(1, math.floor(7 * iRate(iAmt(), IR.Fireflies.lo, IR.Fireflies.hi) + 0.5))
            for n = 0, 5 do
                task.delay(1.6 + n * 0.7, function()
                    if _swarm.active and _ff.burst then _ff.burst:Emit(em) end
                end)
            end
        end)
    end)()
    local stopLeafDevil
    ;(function()
        local RING_N   = 11
        local DEV_H    = 25
        local DEV_R    = 9.5
        local DEV_LIFE = 14
        local DEV_HOLD = 9.6
        local DEV_DOWN = Vec(0, -80, 0)
        local RELEASE  = Vec(-8, -3.5, 3)
        local RING_TEX  = { TX_LEAFA, TX_LEAFB, TX_LEAFA, TX_LEAFB, TX_LEAFC }
        local RING_GLOW = { 0.62, 0.07, 0.6, 0.08, 0.1 }
        local RING_COL  = {
            ColorSequence.new({ CSK(0, Color3.fromRGB(226,190,128)), CSK(0.45, Color3.fromRGB(255,246,200)),
                                CSK(1, Color3.fromRGB(190,132,62)) }),
            ColorSequence.new({ CSK(0, Color3.fromRGB(160,80,52)), CSK(1, Color3.fromRGB(104,44,30)) }),
            ColorSequence.new({ CSK(0, Color3.fromRGB(230,222,176)), CSK(0.5, Color3.fromRGB(255,254,228)),
                                CSK(1, Color3.fromRGB(200,180,122)) }),
            ColorSequence.new({ CSK(0, Color3.fromRGB(156,74,255)), CSK(1, Color3.fromRGB(106,46,200)) }),
            ColorSequence.new({ CSK(0, Color3.fromRGB(196,156,112)), CSK(1, Color3.fromRGB(146,110,74)) }),
        }
        local RING_SIZE = NumberSequence.new({ NSK(0, 0.56, 0.14), NSK(1, 0.46, 0.1) })
        local RING_TR   = NumberSequence.new({ NSK(0, 1), NSK(0.07, 0.05), NSK(0.78, 0.4), NSK(1, 1) })
        local RING_SQ   = NumberSequence.new({ NSK(0, -0.6), NSK(0.25, 0.16), NSK(0.5, -0.6),
                                               NSK(0.75, 0.16), NSK(1, -0.6) })
        local SKIRT_SIZE = NumberSequence.new({ NSK(0, 1.5), NSK(1, 4.4) })
        local SKIRT_TR   = NumberSequence.new({ NSK(0, 1), NSK(0.22, 0.82), NSK(0.75, 0.92), NSK(1, 1) })
        local LEAF_SIZE  = NumberSequence.new({ NSK(0, 0.52, 0.12), NSK(1, 0.44, 0.1) })
        local LEAF_TR    = NumberSequence.new({ NSK(0, 1), NSK(0.1, 0.08), NSK(0.75, 0.42), NSK(1, 1) })
        local DUCK_TI = TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
        local BACK_TI = TweenInfo.new(2.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
        local _dev = { folder = nil, conn = nil }
        local _devRp = RaycastParams.new()
        _devRp.FilterType = Enum.RaycastFilterType.Exclude
        local function ambientRate(mul, ti)
            local I = iAmt()
            for _, L in _partLayers do
                if L.emitter and L.tag == nil then
                    if L.rateTween then L.rateTween:Cancel() end
                    L.rateMul = mul
                    local tw = TweenService:Create(L.emitter, ti, { Rate = layerRate(L, I) * mul })
                    L.rateTween = tw
                    tw:Play()
                end
            end
        end
        stopLeafDevil = function()
            if _dev.conn then _dev.conn:Disconnect(); _dev.conn = nil end
            if _dev.folder then pcall(function() _dev.folder:Destroy() end); _dev.folder = nil end
            if _dev.rings then
                _dev.rings = nil
                ambientRate(1, BACK_TI)
            end
        end
        local function mkRing(folder, cf, i, r)
            local p = mkBoltPart(WHITE, 1, Vec(0.3, 0.3, 0.3), cf, folder)
            local k = ((i - 1) % 5) + 1
            local glow = RING_GLOW[k]
            local e = Instance.new("ParticleEmitter")
            e:SetAttribute("WX_Custom", true)
            e.Texture = RING_TEX[k]
            e.Color = RING_COL[k]
            e.LightEmission = glow; e.LightInfluence = 1 - glow
            e.Size = RING_SIZE; e.Transparency = RING_TR; e.Squash = RING_SQ
            e.Rate = 0
            e.Lifetime = NumberRange.new(2, 3)
            e.Speed = NumberRange.new(5, 9)
            e.SpreadAngle = Vector2.new(8, 8)
            e.Rotation = NumberRange.new(-180, 180)
            e.RotSpeed = NumberRange.new(-300, 300)
            e.Acceleration = Vec(0, 2.6, 0)
            e.Drag = 1.4
            e.EmissionDirection = ND.Front
            e.Parent = p
            return { part = p, e = e, r = r }
        end
        registerSpectacle("LeafDevil", 1, function()
            if not Config.Weather or _curType ~= "Autumn" or _dev.conn then return end
            local cam = Camera; if not cam then return end
            local camPos = cam.CFrame.Position
            local bearing = math.random() * 6.283
            local dist = 18 + math.random() * 12
            local cx = camPos.X + math.cos(bearing) * dist
            local cz = camPos.Z + math.sin(bearing) * dist
            local ch = lp and lp.Character
            if ch then
                _devRp.FilterDescendantsInstances = { getFolder(), ch }
            else
                _devRp.FilterDescendantsInstances = { getFolder() }
            end
            local hit = Workspace:Raycast(Vec(cx, camPos.Y + 6, cz), DEV_DOWN, _devRp)
            local gy = camPos.Y - 4.6
            if hit then gy = hit.Position.Y end
            local folder = Instance.new("Folder")
            folder.Name = "_wxDevil"; folder:SetAttribute("WX_Custom", true)
            folder.Parent = getFolder()
            _dev.folder = folder
            local em = iRate(iAmt(), IR.Autumn.lo, IR.Autumn.hi)
            local seed = CFrame.new(cx, gy + 1, cz)
            local rings = table.create(RING_N)
            for i = 1, RING_N do
                local h = 0.8 + (i - 1) * (DEV_H - 0.8) / (RING_N - 1)
                local u = h / DEV_H
                local r = DEV_R * (0.15 + 0.85 * u ^ 0.9)
                if u > 0.72 then r = r * (1 + (u - 0.72) * 1.5) end
                local R = mkRing(folder, seed, i, r)
                R.h = h
                R.ph = (i - 1) * 0.62
                R.t0 = 0.12 * (i - 1)
                R.rate = (10 + 34 * (r / DEV_R)) * em
                rings[i] = R
            end
            _dev.rings = rings
            local foot = mkBoltPart(WHITE, 1, Vec(6.5, 0.4, 6.5), CFrame.new(cx, gy + 0.35, cz), folder)
            local leafSkirt = Instance.new("ParticleEmitter")
            leafSkirt:SetAttribute("WX_Custom", true)
            leafSkirt.Texture = TX_LEAFA
            leafSkirt.Color = RING_COL[1]
            leafSkirt.LightEmission = 0.3; leafSkirt.LightInfluence = 0.7
            leafSkirt.Size = LEAF_SIZE; leafSkirt.Transparency = LEAF_TR; leafSkirt.Squash = RING_SQ
            leafSkirt.Rate = 0
            leafSkirt.Lifetime = NumberRange.new(1.2, 2.2)
            leafSkirt.Speed = NumberRange.new(2, 5)
            leafSkirt.SpreadAngle = Vector2.new(70, 70)
            leafSkirt.Rotation = NumberRange.new(-180, 180)
            leafSkirt.RotSpeed = NumberRange.new(-260, 260)
            leafSkirt.Acceleration = Vec(0, 1.2, 0)
            leafSkirt.Drag = 2.2
            leafSkirt.EmissionDirection = ND.Top
            leafSkirt.Parent = foot
            local dustSkirt = Instance.new("ParticleEmitter")
            dustSkirt:SetAttribute("WX_Custom", true)
            dustSkirt.Texture = TX_DUST
            dustSkirt.Color = ColorSequence.new(Color3.fromRGB(190,166,130), Color3.fromRGB(140,118,90))
            dustSkirt.LightEmission = 0.04; dustSkirt.LightInfluence = 0.96
            dustSkirt.Size = SKIRT_SIZE; dustSkirt.Transparency = SKIRT_TR
            dustSkirt.Rate = 0
            dustSkirt.Lifetime = NumberRange.new(0.9, 1.4)
            dustSkirt.Speed = NumberRange.new(1, 2.6)
            dustSkirt.SpreadAngle = Vector2.new(85, 85)
            dustSkirt.RotSpeed = NumberRange.new(-25, 25)
            dustSkirt.Acceleration = Vec(0, 1, 0)
            dustSkirt.Drag = 2.4
            dustSkirt.EmissionDirection = ND.Top
            dustSkirt.Parent = foot
            local scuff = mkBoltPart(WHITE, 1, Vec(15, 0.08, 15), CFrame.new(cx, gy + 0.05, cz), folder)
            scuff.Material = Enum.Material.SmoothPlastic
            local sd = Instance.new("Decal")
            sd:SetAttribute("WX_Custom", true)
            sd.Face = ND.Top; sd.Texture = TX_DUST
            sd.Color3 = Color3.fromRGB(96, 74, 52); sd.Transparency = 0.55
            sd.Parent = scuff
            ambientRate(0.45, DUCK_TI)
            local t0 = tick()
            local ang, px, pz = 0, cx, cz
            local lean = math.random() * 6.283
            _dev.conn = RunService.Heartbeat:Connect(function(dt)
                if not folder.Parent then stopLeafDevil() return end
                local t = tick() - t0
                if t > DEV_LIFE then stopLeafDevil() return end
                ang = ang + dt * 2.7
                px = px + Wind.x * dt * 0.75
                pz = pz + Wind.z * dt * 0.75
                local lx = math.cos(lean + t * 0.21) * 0.13
                local lz = math.sin(lean + t * 0.17) * 0.13
                local climb = math.min(t / 1.5, 1)
                local release = t > DEV_HOLD
                for i = 1, RING_N do
                    local R = rings[i]
                    local h = R.h * climb
                    local th = ang + R.ph
                    local cs, sn = math.cos(th), math.sin(th)
                    local r = R.r * (0.55 + 0.45 * climb)
                    local p = Vec(px + lx * h + cs * r, gy + 0.35 + h, pz + lz * h + sn * r)
                    R.part.CFrame = CFrame.lookAt(p, p + Vec(-sn, 0.3, cs))
                    if release then
                        R.e.Rate = 0
                        R.e.Acceleration = RELEASE
                    else
                        R.e.Rate = R.rate * math.clamp((t - R.t0) / 0.6, 0, 1)
                    end
                end
                foot.CFrame = CFrame.new(px, gy + 0.35, pz)
                scuff.CFrame = CFrame.new(px, gy + 0.05, pz) * CFrame.Angles(0, t * 0.35, 0)
                if release then
                    leafSkirt.Rate = 0
                    dustSkirt.Rate = 0
                    leafSkirt.Acceleration = RELEASE
                    sd.Transparency = math.min(0.55 + (t - DEV_HOLD) * 0.35, 1)
                    if t > DEV_HOLD + 0.1 and _dev.rings then
                        _dev.rings = nil
                        ambientRate(1, BACK_TI)
                    end
                else
                    local k = math.min(t / 0.8, 1)
                    leafSkirt.Rate = 26 * em * k
                    dustSkirt.Rate = 15 * em * k
                    sd.Transparency = 0.95 - 0.4 * math.min(t / 1.2, 1)
                end
            end)
        end)
    end)()
    local puddlesRefresh, puddlesIntensity, stopPuddles
    ;(function()
        local TXP_RING = "rbxassetid://17738857765"
        local TXP_DROP = "rbxassetid://14964503448"
        local TXP_STAR = "rbxassetid://17726943419"
        local WATER_C  = Color3.fromRGB(22, 28, 37)
        local SHEEN_C  = Color3.fromRGB(150, 175, 205)
        local FOAM_C   = Color3.fromRGB(236, 246, 255)
        local PUD_DOWN = Vec(0, -90, 0)
        local FIT_DOWN = Vec(0, -4.2, 0)
        local HALF_PI  = 1.5707963
        local FIT_TOL  = 1.5
        local FIT_R    = 1.09
        local FIT_K    = { 1, 0.66, 0.44, 0.28 }
        local FIT_C = {
            1, 0, 0.70711, 0.70711, 0, 1, -0.70711, 0.70711,
            -1, 0, -0.70711, -0.70711, 0, -1, 0.70711, -0.70711,
        }
        local FIT_Q = {
            1, 1, 0.333, 1, -0.333, 1, -1, 1,
            -1, 0.333, -1, -0.333, -1, -1, -0.333, -1,
            0.333, -1, 1, -1, 1, -0.333, 1, 0.333,
        }
        local CELL     = 32
        local HALF     = 16
        local NOISE_F  = 0.018
        local POOL_F   = 0.74
        local TILE_K   = 1 / 470
        local K_CAP    = 48
        local MAX_REC  = 96
        local _pud = { folder = nil, conn = nil }
        local _plist = {}
        local _pmap = {}
        local _pfail = {}
        local _cd = { i = 0, j = 0, key = 0, kind = 0, cx = 0, cz = 0, gy = 0 }
        local _units, _uIdx = {}, 1
        local _prints, _pIdx = {}, 1
        local _pstates = {}
        local _abuf = {}
        local _prp = RaycastParams.new()
        _prp.FilterType = Enum.RaycastFilterType.Exclude
        local _pfilter = {}
        local _fieldToken, _rebuild = 0, false
        local _int, _thr, _placeR, _seed = 1, 0, 50, 0
        local _wTr, _wRf = 0.35, 0.16
        local _homeI, _homeJ = 1e9, 1e9
        local function pudActive()
            if not Config.Weather or not cfg("WeatherPuddles", false) then
                return false
            end
            if _curType == "Rain" then
                return true
            end
            return cfg("WeatherStorm", false)
        end
        local function pudIntensity()
            return iAmt()
        end
        local function mkFlatPart(size, cf, parent)
            local p = Instance.new("Part")
            p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false
            p.CastShadow = false; p.Transparency = 1
            p.Size = size; p.CFrame = cf
            p:SetAttribute("WX_Custom", true)
            p.Parent = parent
            return p
        end
        local function mkSlab(d1, d2, cf, parent, sheen)
            local p = Instance.new("Part")
            p.Shape = Enum.PartType.Cylinder
            p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false
            p.CastShadow = false; p.Material = Enum.Material.SmoothPlastic
            if sheen then
                p.Color = SHEEN_C; p.Transparency = 0.74; p.Reflectance = 0.38
            else
                p.Color = WATER_C; p.Transparency = _wTr; p.Reflectance = _wRf
                p:SetAttribute("WX_W", true)
            end
            p.Size = Vec(0.05, d1, d2)
            p.CFrame = cf
            p:SetAttribute("WX_Custom", true)
            p.Parent = parent
        end
        local function mkTile(w, l, cf, parent)
            local p = Instance.new("Part")
            p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false
            p.CastShadow = false; p.Material = Enum.Material.SmoothPlastic
            p.Color = WATER_C; p.Transparency = _wTr; p.Reflectance = _wRf
            p.Size = Vec(w, 0.05, l)
            p.CFrame = cf
            p:SetAttribute("WX_Custom", true)
            p:SetAttribute("WX_W", true)
            p.Parent = parent
        end
        local function fitSlab(cx, cz, d1, d2, yaw, gy)
            local cs, sn = math.cos(yaw), math.sin(yaw)
            for k = 1, 4 do
                local f = FIT_K[k] * FIT_R * 0.5
                local a, b = d1 * f, d2 * f
                local ok, gx, gz, n = true, 0, 0, 0
                for i = 1, 8 do
                    local ct, st = FIT_C[i * 2 - 1], FIT_C[i * 2]
                    local ox = sn * b * st - cs * a * ct
                    local oz = sn * a * ct + cs * b * st
                    local h = Workspace:Raycast(Vec(cx + ox, gy + 2, cz + oz), FIT_DOWN, _prp)
                    local sup = false
                    if h and h.Normal.Y >= 0.86 and h.Instance.CanCollide then
                        local dy = h.Position.Y - gy
                        sup = dy < FIT_TOL and dy > -FIT_TOL
                    end
                    if sup then
                        gx, gz, n = gx + ox, gz + oz, n + 1
                    else
                        ok = false
                    end
                end
                if ok then
                    return cx, cz, d1 * FIT_K[k], d2 * FIT_K[k]
                end
                if n == 0 then
                    return nil
                end
                cx, cz = cx + gx / n * 0.5, cz + gz / n * 0.5
            end
            return nil
        end
        local function fitRect(cx, cz, w, l, gy)
            for k = 1, 4 do
                local f = FIT_K[k] * FIT_R * 0.5
                local a, b = w * f, l * f
                local ok = true
                for i = 1, 12 do
                    local h = Workspace:Raycast(
                        Vec(cx + FIT_Q[i * 2 - 1] * a, gy + 2, cz + FIT_Q[i * 2] * b), FIT_DOWN, _prp)
                    local sup = false
                    if h and h.Normal.Y >= 0.86 and h.Instance.CanCollide then
                        local dy = h.Position.Y - gy
                        sup = dy < FIT_TOL and dy > -FIT_TOL
                    end
                    if not sup then
                        ok = false
                        break
                    end
                end
                if ok then
                    return w * FIT_K[k], l * FIT_K[k]
                end
            end
            return nil
        end
        local function mkHost(m, cf, size, k, I)
            local host = mkFlatPart(size, cf, m)
            local e = Instance.new("ParticleEmitter")
            e:SetAttribute("WX_Custom", true)
            e.Texture = TXP_RING
            e.Orientation = Enum.ParticleOrientation.VelocityPerpendicular
            e.EmissionDirection = Enum.NormalId.Top
            e.Speed = NumberRange.new(0.05, 0.05)
            e.Rate = (2.1 + 4.2 * I) * k
            e.Lifetime = NumberRange.new(1.1, 1.6)
            e.Size = NumberSequence.new({ NSK(0, 0.12), NSK(1, 4.4) })
            e.Transparency = NumberSequence.new({ NSK(0, 0.9), NSK(0.14, 0.12), NSK(0.62, 0.5), NSK(1, 1) })
            e.Color = ColorSequence.new(Color3.fromRGB(215, 234, 250))
            e.LightEmission = 0.45
            e.Parent = host
            local g = Instance.new("ParticleEmitter")
            g:SetAttribute("WX_Custom", true)
            g.Texture = TXP_STAR
            g.EmissionDirection = Enum.NormalId.Top
            g.Speed = NumberRange.new(0, 0)
            g.Rate = 0.8 * I * k
            g.Lifetime = NumberRange.new(0.16, 0.28)
            g.Size = NumberSequence.new({ NSK(0, 0.06), NSK(0.4, 0.3), NSK(1, 0.04) })
            g.Transparency = NumberSequence.new({ NSK(0, 0.6), NSK(1, 1) })
            g.Color = ColorSequence.new(Color3.fromRGB(240, 248, 255))
            g.LightEmission = 0.5
            g.Parent = host
            return e, g
        end
        local function mkPuddle(cd, span, nslab, I)
            local x, z, gy = cd.cx, cd.cz, cd.gy
            local m = Instance.new("Model")
            m.Name = "_pud"
            local px = x + (math.random() - 0.5) * CELL * 0.16
            local pz = z + (math.random() - 0.5) * CELL * 0.16
            local spine = math.random() * 6.28318
            local sx, sz = math.cos(spine), math.sin(spine)
            local lx, lz = -sz, sx
            local yaw0 = math.pi - spine
            local slabs, rmax = {}, 0
            local s0, s1, l0, l1 = 1e9, -1e9, 1e9, -1e9
            local function place(cx, cz, d1, d2, yaw, yoff, sheen)
                local fx, fz, fd1, fd2 = fitSlab(cx, cz, d1, d2, yaw, gy)
                if not fx then
                    return false
                end
                local rad = math.max(fd1, fd2) * 0.5
                local ex, ez = fx - x, fz - z
                if math.abs(ex) + rad > HALF or math.abs(ez) + rad > HALF then
                    return false
                end
                mkSlab(fd1, fd2, CFrame.new(fx, gy + yoff, fz)
                    * CFrame.Angles(0, yaw, 0) * CFrame.Angles(0, 0, HALF_PI), m, sheen)
                if not sheen then
                    slabs[#slabs + 1] = {
                        cx = fx, cz = fz,
                        cs = math.cos(yaw), sn = math.sin(yaw),
                        ia = 2 / fd1, ib = 2 / fd2, rect = false,
                    }
                    rmax = math.max(rmax, math.sqrt(ex * ex + ez * ez) + rad)
                end
                local dx, dz = fx - px, fz - pz
                local t, lat = dx * sx + dz * sz, dx * lx + dz * lz
                local ins = math.min(fd1, fd2) * 0.375
                s0, s1 = math.min(s0, t - ins), math.max(s1, t + ins)
                l0, l1 = math.min(l0, lat - ins), math.max(l1, lat + ins)
                return true
            end
            if not place(px, pz, span, span * 0.62, yaw0, 0.002, false) then
                m:Destroy()
                return false
            end
            for i = 1, nslab do
                local t = ((i - 0.5) / nslab - 0.5) * span * 0.62 + (math.random() - 0.5) * span * 0.14
                local lat = (math.random() - 0.5) * span * 0.24
                local d1 = span * (0.34 + math.random() * 0.3)
                place(px + sx * t + lx * lat, pz + sz * t + lz * lat,
                    d1, d1 * (0.6 + math.random() * 0.34),
                    yaw0 + (math.random() - 0.5) * 1.6, 0.008 + i * 0.004, false)
            end
            for _ = 1, 2 do
                local t = (math.random() - 0.5) * span * 0.5
                local lat = (math.random() - 0.5) * span * 0.16
                local d1 = span * (0.14 + math.random() * 0.18)
                place(px + sx * t + lx * lat, pz + sz * t + lz * lat,
                    d1, d1 * (0.4 + math.random() * 0.35), math.random() * math.pi, 0.036, true)
            end
            s0, s1 = math.max(s0, span * -0.425), math.min(s1, span * 0.425)
            l0, l1 = math.max(l0, span * -0.225), math.min(l1, span * 0.225)
            local hw, hl = math.max(s1 - s0, 1), math.max(l1 - l0, 1)
            local hm, hn = (s0 + s1) * 0.5, (l0 + l1) * 0.5
            local k = (span / 26) ^ 1.5 * hw * hl / (span * span * 0.3825)
            local e, g = mkHost(m, CFrame.new(px + sx * hm + lx * hn, gy + 0.1, pz + sz * hm + lz * hn)
                * CFrame.Angles(0, -spine, 0), Vec(hw, 0.05, hl), k, I)
            m.Parent = _pud.folder
            local P = {
                cx = x, cz = z, top = gy + 0.06, r2 = rmax * rmax, k = k,
                key = cd.key, ci = cd.i, cj = cd.j, kind = cd.kind,
                slabs = slabs, ring = e, glint = g, model = m,
            }
            _plist[#_plist + 1] = P
            _pmap[cd.key] = P
            return true
        end
        local function mkFloodCell(cd, I)
            local gy = cd.gy
            local m = Instance.new("Model")
            m.Name = "_pud"
            local slabs = {}
            local area, rmax = 0, 0
            local x0, x1, z0, z1 = 1e9, -1e9, 1e9, -1e9
            for a = 0, 1 do
                for b = 0, 1 do
                    local scx = cd.cx + (a - 0.5) * HALF
                    local scz = cd.cz + (b - 0.5) * HALF
                    local fw, fl = fitRect(scx, scz, HALF, HALF, gy)
                    if fw then
                        mkTile(fw, fl, CFrame.new(scx, gy + 0.002, scz), m)
                        slabs[#slabs + 1] = {
                            cx = scx, cz = scz, cs = 1, sn = 0,
                            ia = 2 / fw, ib = 2 / fl, rect = true,
                        }
                        area = area + fw * fl
                        rmax = math.max(rmax, 0.7072 * (HALF + fw))
                        x0 = math.min(x0, scx - fw * 0.5)
                        x1 = math.max(x1, scx + fw * 0.5)
                        z0 = math.min(z0, scz - fl * 0.5)
                        z1 = math.max(z1, scz + fl * 0.5)
                    end
                end
            end
            if area <= 0 then
                m:Destroy()
                return false
            end
            if area > CELL * CELL * 0.98 and math.random() < 0.45 then
                local d1 = CELL * (0.15 + math.random() * 0.1)
                local sx = cd.cx + (math.random() - 0.5) * CELL * 0.4
                local sz = cd.cz + (math.random() - 0.5) * CELL * 0.4
                local yaw = math.random() * math.pi
                local fx, fz, fd1, fd2 = fitSlab(sx, sz, d1, d1 * (0.4 + math.random() * 0.35), yaw, gy)
                if fx and math.abs(fx - cd.cx) + fd1 * 0.5 < HALF
                    and math.abs(fz - cd.cz) + fd1 * 0.5 < HALF then
                    mkSlab(fd1, fd2, CFrame.new(fx, gy + 0.036, fz)
                        * CFrame.Angles(0, yaw, 0) * CFrame.Angles(0, 0, HALF_PI), m, true)
                end
            end
            local hw, hl = math.max(x1 - x0, 1), math.max(z1 - z0, 1)
            local e, g = mkHost(m, CFrame.new((x0 + x1) * 0.5, gy + 0.1, (z0 + z1) * 0.5),
                Vec(hw, 0.05, hl), area * TILE_K, I)
            m.Parent = _pud.folder
            local P = {
                cx = cd.cx, cz = cd.cz, top = gy + 0.06, r2 = rmax * rmax, k = area * TILE_K,
                key = cd.key, ci = cd.i, cj = cd.j, kind = cd.kind,
                slabs = slabs, ring = e, glint = g, model = m,
            }
            _plist[#_plist + 1] = P
            _pmap[cd.key] = P
            return true
        end
        local function mkSplashUnit(parent)
            local p = mkFlatPart(Vec(1.1, 0.1, 1.1), CFrame.new(0, -900, 0), parent)
            local ring = Instance.new("ParticleEmitter")
            ring:SetAttribute("WX_Custom", true)
            ring.Texture = TXP_RING
            ring.Orientation = Enum.ParticleOrientation.VelocityPerpendicular
            ring.EmissionDirection = Enum.NormalId.Top
            ring.Speed = NumberRange.new(0.05, 0.05)
            ring.Rate = 0
            ring.Lifetime = NumberRange.new(0.45, 0.7)
            ring.Size = NumberSequence.new({ NSK(0, 0.1), NSK(1, 4) })
            ring.Transparency = NumberSequence.new({ NSK(0, 0.75), NSK(0.1, 0.05), NSK(0.5, 0.4), NSK(1, 1) })
            ring.Color = ColorSequence.new(Color3.fromRGB(230, 244, 255))
            ring.LightEmission = 0.5
            ring.Parent = p
            local foam = Instance.new("ParticleEmitter")
            foam:SetAttribute("WX_Custom", true)
            foam.Texture = TX_SNOWSOFT
            foam.Orientation = Enum.ParticleOrientation.VelocityParallel
            foam.EmissionDirection = Enum.NormalId.Top
            foam.Rate = 0
            foam.SpreadAngle = Vector2.new(56, 56)
            foam.Acceleration = Vec(0, -80, 0)
            foam.Lifetime = NumberRange.new(0.3, 0.6)
            foam.Size = NumberSequence.new({ NSK(0, 0.6, 0.22), NSK(1, 0.2) })
            foam.Squash = NumberSequence.new({ NSK(0, 1.5), NSK(1, 0.4) })
            foam.Speed = NumberRange.new(6, 12)
            foam.Color = ColorSequence.new(FOAM_C)
            foam.Transparency = NumberSequence.new({ NSK(0, 0.18), NSK(0.7, 0.45), NSK(1, 1) })
            foam.LightEmission = 0.18
            foam.Parent = p
            local drops = Instance.new("ParticleEmitter")
            drops:SetAttribute("WX_Custom", true)
            drops.Texture = TXP_DROP
            drops.Orientation = Enum.ParticleOrientation.VelocityParallel
            drops.EmissionDirection = Enum.NormalId.Top
            drops.Rate = 0
            drops.SpreadAngle = Vector2.new(52, 52)
            drops.Acceleration = Vec(0, -80, 0)
            drops.Lifetime = NumberRange.new(0.32, 0.62)
            drops.Size = NumberSequence.new({ NSK(0, 0.55, 0.18), NSK(1, 0.28) })
            drops.Speed = NumberRange.new(6, 12)
            drops.Color = ColorSequence.new(Color3.fromRGB(210, 232, 250))
            drops.Transparency = NumberSequence.new({ NSK(0, 0.1), NSK(0.75, 0.3), NSK(1, 1) })
            drops.LightEmission = 0.25
            drops.Parent = p
            local mist = Instance.new("ParticleEmitter")
            mist:SetAttribute("WX_Custom", true)
            mist.Texture = TX_SNOWSOFT
            mist.EmissionDirection = Enum.NormalId.Top
            mist.Rate = 0
            mist.Speed = NumberRange.new(1.2, 2.8)
            mist.SpreadAngle = Vector2.new(48, 48)
            mist.Lifetime = NumberRange.new(0.3, 0.5)
            mist.Size = NumberSequence.new({ NSK(0, 0.5), NSK(1, 1.6) })
            mist.Transparency = NumberSequence.new({ NSK(0, 0.75), NSK(1, 1) })
            mist.Color = ColorSequence.new(FOAM_C)
            mist.LightEmission = 0.1
            mist.Parent = p
            return { part = p, ring = ring, foam = foam, drops = drops, mist = mist }
        end
        local function fireSplash(x, y, z, speed)
            local u = _units[_uIdx]
            if not u then
                return
            end
            _uIdx = (_uIdx % #_units) + 1
            local s = math.clamp(speed, 4, 36)
            u.part.CFrame = CFrame.new(x, y + 0.12, z)
            u.ring.Size = NumberSequence.new({ NSK(0, 0.1), NSK(1, 3.2 + s * 0.18) })
            local lo, hi = 5 + s * 0.38, 8 + s * 0.6
            u.foam.Speed = NumberRange.new(lo, hi)
            u.drops.Speed = NumberRange.new(lo * 0.8, hi * 0.9)
            u.ring:Emit(2)
            u.foam:Emit(9 + math.floor(s * 0.75))
            u.drops:Emit(4 + math.floor(s * 0.3))
            u.mist:Emit(3)
        end
        local function dropPrint(x, y, z, heading, side)
            local p = _prints[_pIdx]
            if not p then
                return
            end
            _pIdx = (_pIdx % #_prints) + 1
            p.CFrame = CFrame.new(x, y + 0.02, z) * CFrame.Angles(0, heading, 0) * CFrame.new(side * 0.4, 0, 0)
            p.Transparency = 0.42
            TweenService:Create(p, TweenInfo.new(2.2), { Transparency = 1 }):Play()
        end
        local function defaultActors()
            local n = 0
            local cam = Camera
            if not cam then
                return _abuf, 0
            end
            local camPos = cam.CFrame.Position
            for _, plr in Players:GetPlayers() do
                local ch = plr.Character
                local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local pos = hrp.Position
                    local dx, dy, dz = pos.X - camPos.X, pos.Y - camPos.Y, pos.Z - camPos.Z
                    if dx * dx + dy * dy + dz * dz < 12100 then
                        local vel = hrp.AssemblyLinearVelocity
                        local grounded = vel.Y > -6 and vel.Y < 6
                        if not grounded then
                            local hum = ch:FindFirstChildOfClass("Humanoid")
                            grounded = hum ~= nil and hum.FloorMaterial ~= Enum.Material.Air
                        end
                        n = n + 1
                        local rec = _abuf[n]
                        if not rec then
                            rec = {}
                            _abuf[n] = rec
                        end
                        rec.key = plr
                        rec.fx, rec.fy, rec.fz = pos.X, pos.Y - 2.9, pos.Z
                        rec.vx, rec.vy, rec.vz = vel.X, vel.Y, vel.Z
                        rec.speed = math.sqrt(vel.X * vel.X + vel.Z * vel.Z)
                        rec.grounded = grounded
                    end
                end
            end
            return _abuf, n
        end
        local actorsProvider = defaultActors
        local function inPuddle(P, x, z)
            local dx, dz = x - P.cx, z - P.cz
            if dx * dx + dz * dz > P.r2 then
                return false
            end
            for _, sl in P.slabs do
                local ux, uz = x - sl.cx, z - sl.cz
                local u = (-ux * sl.cs + uz * sl.sn) * sl.ia
                local v = (ux * sl.sn + uz * sl.cs) * sl.ib
                if sl.rect then
                    if u > -1 and u < 1 and v > -1 and v < 1 then
                        return true
                    end
                elseif u * u + v * v <= 1 then
                    return true
                end
            end
            return false
        end
        local function fieldAt(wx, wz)
            return math.noise(wx * NOISE_F, wz * NOISE_F, _seed)
        end
        local function floodThr(I)
            if I <= 1 then
                return -0.4043 * (I - 0.3)
            end
            if I <= 1.6 then
                return -0.283 - 0.3417 * (I - 1)
            end
            return -0.488 - 1.405 * (I - 1.6)
        end
        local function refreshParams()
            local I = pudIntensity()
            _int = I
            _thr = floodThr(I)
            _placeR = 46 + 20 * I
            _wTr = 0.46 - 0.11 * I
            _wRf = 0.11 + 0.05 * I
        end
        local function cellKind(i, j)
            local ccx, ccz = (i + 0.5) * CELL, (j + 0.5) * CELL
            if fieldAt(ccx, ccz) <= _thr then
                return 0
            end
            if fieldAt(ccx + CELL, ccz) > _thr and fieldAt(ccx - CELL, ccz) > _thr
                and fieldAt(ccx, ccz + CELL) > _thr and fieldAt(ccx, ccz - CELL) > _thr then
                return 2
            end
            return 1
        end
        local function placeCell(i, j, key, kind, by)
            local folder = _pud.folder
            if not folder then
                return false
            end
            local ccx, ccz = (i + 0.5) * CELL, (j + 0.5) * CELL
            _pfilter[1] = folder
            local ch = lp and lp.Character
            if ch then
                _pfilter[2] = ch
            else
                _pfilter[2] = folder
            end
            _prp.FilterDescendantsInstances = _pfilter
            local hit = Workspace:Raycast(Vec(ccx, by + 6, ccz), PUD_DOWN, _prp)
            if not hit or hit.Normal.Y < 0.94 then
                return false
            end
            local inst = hit.Instance
            if not inst.CanCollide then
                return false
            end
            if inst:IsA("Terrain") and hit.Material == Enum.Material.Water then
                return false
            end
            local cd = _cd
            cd.i, cd.j, cd.key, cd.kind = i, j, key, kind
            cd.cx, cd.cz, cd.gy = ccx, ccz, hit.Position.Y
            if kind == 2 then
                return mkFloodCell(cd, _int)
            end
            local nslab = 5
            if _int > 1.15 then
                nslab = 4
            end
            return mkPuddle(cd, CELL * POOL_F * (0.82 + 0.24 * math.random()), nslab, _int)
        end
        local function tryCell(i, j, bx, bz, by)
            local key = (i + 32768) * 65536 + j + 32768
            if _pmap[key] or _pfail[key] then
                return false
            end
            local dx = (i + 0.5) * CELL - bx
            local dz = (j + 0.5) * CELL - bz
            if dx * dx + dz * dz > _placeR * _placeR then
                return false
            end
            local kind = cellKind(i, j)
            if kind == 0 then
                return false
            end
            if not placeCell(i, j, key, kind, by) then
                _pfail[key] = true
            end
            return true
        end
        local function maintain(bx, by, bz)
            local ci, cj = math.floor(bx / CELL), math.floor(bz / CELL)
            local budget = 8
            if ci ~= _homeI or cj ~= _homeJ then
                _homeI, _homeJ = ci, cj
                table.clear(_pfail)
            end
            if _rebuild then
                _rebuild = false
                refreshParams()
                table.clear(_pfail)
                budget = 18
                for _, P in _plist do
                    for _, d in P.model:GetChildren() do
                        if d:GetAttribute("WX_W") then
                            d.Transparency = _wTr
                            d.Reflectance = _wRf
                        end
                    end
                end
                local n = 1
                while n <= #_plist do
                    local P = _plist[n]
                    if P.kind == cellKind(P.ci, P.cj) then
                        n = n + 1
                    else
                        _pmap[P.key] = nil
                        P.model:Destroy()
                        _plist[n] = _plist[#_plist]
                        _plist[#_plist] = nil
                    end
                end
            end
            local cull = _placeR + CELL * 0.75
            local cull2 = cull * cull
            local n = 1
            while n <= #_plist do
                local P = _plist[n]
                local dx, dz = P.cx - bx, P.cz - bz
                if dx * dx + dz * dz > cull2 then
                    _pmap[P.key] = nil
                    P.model:Destroy()
                    _plist[n] = _plist[#_plist]
                    _plist[#_plist] = nil
                else
                    n = n + 1
                end
            end
            local nr = math.ceil(_placeR / CELL)
            for ring = 0, nr do
                for di = -ring, ring do
                    for dj = -ring, ring do
                        if budget > 0 and #_plist < MAX_REC
                            and math.max(math.abs(di), math.abs(dj)) == ring
                            and tryCell(ci + di, cj + dj, bx, bz, by) then
                            budget = budget - 1
                        end
                    end
                end
            end
            local ksum = 0
            for _, P in _plist do
                ksum = ksum + P.k
            end
            local damp = 1
            if ksum > K_CAP then
                damp = K_CAP / ksum
            end
            local rr = (2.1 + 4.2 * _int) * damp
            local gr = 0.8 * _int * damp
            for _, P in _plist do
                P.ring.Rate = rr * P.k
                P.glint.Rate = gr * P.k
            end
        end
        local function tick20(now, doMaintain)
            local bx, by, bz
            local ch = lp and lp.Character
            local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
            if hrp then
                local p = hrp.Position
                bx, by, bz = p.X, p.Y, p.Z
            elseif Camera then
                local p = Camera.CFrame.Position
                bx, by, bz = p.X, p.Y, p.Z
            else
                return
            end
            if doMaintain then
                maintain(bx, by, bz)
            end
            local list, n = actorsProvider()
            for ai = 1, n do
                local a = list[ai]
                local st = _pstates[a.key]
                if not st then
                    st = { last = 0, wet = 0, side = 1, prevVy = 0 }
                    _pstates[a.key] = st
                end
                local sp = a.speed
                local pud = nil
                for _, P in _plist do
                    if math.abs(a.fy - P.top) < 4.5 and inPuddle(P, a.fx, a.fz) then
                        pud = P
                        break
                    end
                end
                local interval = math.clamp(3.6 / math.max(sp, 1), 0.15, 0.32)
                if pud and a.grounded and sp > 2.2 then
                    local landing = st.prevVy < -25
                    if landing or now - st.last >= interval then
                        st.last = now
                        st.side = -st.side
                        local fs = sp
                        if landing then
                            fs = 36
                        end
                        fireSplash(a.fx, pud.top, a.fz, fs)
                        st.wet = now + 1.7
                    end
                elseif not pud and now < st.wet and a.grounded and sp > 2.2 then
                    if now - st.last >= interval then
                        st.last = now
                        st.side = -st.side
                        dropPrint(a.fx, a.fy, a.fz, math.atan2(-a.vx, -a.vz), st.side)
                    end
                end
                st.prevVy = a.vy
            end
        end
        local function startPuddles()
            if _pud.conn then
                return
            end
            local folder = Instance.new("Folder")
            folder.Name = "_wxPuddles"
            folder:SetAttribute("WX_Custom", true)
            folder.Parent = getFolder()
            _pud.folder = folder
            _plist = {}
            _pmap = {}
            table.clear(_pfail)
            _seed = math.random() * 512
            _homeI, _homeJ = 1e9, 1e9
            refreshParams()
            _units, _uIdx = {}, 1
            for i = 1, 8 do
                _units[i] = mkSplashUnit(folder)
            end
            _prints, _pIdx = {}, 1
            for i = 1, 12 do
                local p = mkFlatPart(Vec(0.6, 0.02, 1.0), CFrame.new(0, -900, 0), folder)
                p.Material = Enum.Material.SmoothPlastic
                p.Color = Color3.fromRGB(14, 18, 25)
                p.Reflectance = 0.18
                _prints[i] = p
            end
            local accum = 0
            local mTick = 0
            _pud.conn = RunService.Heartbeat:Connect(function(dt)
                accum = accum + dt
                if accum < 0.05 then
                    return
                end
                accum = 0
                if not pudActive() then
                    return
                end
                mTick = mTick + 1
                local doMaintain = false
                if mTick >= 14 then
                    mTick = 0
                    doMaintain = true
                end
                pcall(tick20, os.clock(), doMaintain)
            end)
        end
        function stopPuddles(hard)
            if _pud.conn then
                _pud.conn:Disconnect()
                _pud.conn = nil
            end
            table.clear(_pstates)
            local folder = _pud.folder
            _pud.folder = nil
            _plist = {}
            _pmap = {}
            table.clear(_pfail)
            _units = {}
            _prints = {}
            _rebuild = false
            if not folder then
                return
            end
            if hard then
                folder:Destroy()
                return
            end
            for _, m in folder:GetDescendants() do
                if m:IsA("ParticleEmitter") then
                    m.Rate = 0
                elseif m:IsA("Part") and m.Transparency < 1 then
                    TweenService:Create(m, TweenInfo.new(1.4, Enum.EasingStyle.Sine), { Transparency = 1 }):Play()
                end
            end
            task.delay(2.2, function()
                pcall(function() folder:Destroy() end)
            end)
        end
        function puddlesRefresh()
            if pudActive() then
                startPuddles()
            else
                stopPuddles(false)
            end
        end
        function puddlesIntensity()
            if not _pud.conn then
                return
            end
            _fieldToken = _fieldToken + 1
            local token = _fieldToken
            task.delay(0.25, function()
                if token ~= _fieldToken then
                    return
                end
                _rebuild = true
            end)
        end
    end)()
    local function startClock()
        if _clockConn then return end
        local accum = 0
        _clockConn = RunService.Heartbeat:Connect(function(dt)
            accum = accum + dt
            if accum < 1 then return end
            accum = 0
            if not Config.Weather or not cfg("WeatherClockDial", false) then return end
            if Config.Visuals then return end
            local cycle = math.max(cfg("WeatherClockCycleMin", 8), 1) * 60
            local t = ((tick() % cycle) / cycle) * 24
            if math.abs(Lighting.ClockTime - t) > 0.02 then
                Lighting.ClockTime = t
            end
        end)
    end
    local function stopClock()
        if _clockConn then _clockConn:Disconnect(); _clockConn = nil end
    end
    local function normType(name)
        if name == "Rain" or name == "BloodMoon" or PRESETS[name] then
            return name
        end
        return "Rain"
    end
    local function applyType(name)
        name = normType(name)
        local prev = _curType
        stopRain(); clearSpecial()
        if #_partLayers > 0 then
            beginLayerFade()
        end
        _curType = name
        if name == "Rain" then
            buildRain(); startRain()
        elseif name == "BloodMoon" then
            buildBloodMoon(); startSpecialAnim()
        else
            local ramp = PRESETS[prev] ~= nil
            local ir = IR[name] or IR_DEF
            for _, spec in PRESETS[name] do
                mkPartLayer(spec, ir, ramp)
            end
        end
        if name == "Fireflies" then startFireflies() else stopFireflies() end
        applyMood(name)
        startAmbient(SOUND[name])
        puddlesRefresh()
    end
    function Weather.setType(name)
        name = normType(name)
        Config.WeatherType = name
        if Config.Weather then applyType(name) end
    end
    local _intensityToken = 0
    function Weather.setIntensity(v)
        Config.WeatherIntensity = math.clamp(v, 0.15, 2)
        if not Config.Weather then return end
        puddlesIntensity()
        local I = Config.WeatherIntensity
        for _, L in _partLayers do
            if L.emitter then
                pcall(tuneLayer, L, I)
                if L.rateTween then L.rateTween:Cancel(); L.rateTween = nil end
                pcall(function() L.emitter.Rate = layerRate(L, I) * L.rateMul end)
            end
        end
        if _special.kind == "BloodMoon" then pcall(tuneBloodMoon, I) end
        if _rays then
            pcall(function()
                _rays.Intensity = 0.10 + 0.18 * I
                _rays.Spread = 1.1 - 0.2 * I
            end)
        end
        _intensityToken = _intensityToken + 1
        local token = _intensityToken
        task.delay(0.12, function()
            if token ~= _intensityToken then return end
            if not Config.Weather then return end
            if _curType == "Rain" then pcall(tuneRain) end
            if _curType then pcall(applyMood, _curType) end
            pcall(refreshFireflies)
        end)
    end
    function Weather.setSoundVolume(v)
        Config.WeatherSoundVolume = v
        if _ambient then pcall(function() _ambient.Volume = v end) end
    end
    function Weather.toggleStorm(on)
        Config.WeatherStorm = on
        if not Config.Weather then return end
        if on then startStorm() else stopStormLoop() end
        puddlesRefresh()
    end
    function Weather.setStormMin(v)
        Config.WeatherStormMin = math.clamp(v, 1, 30)
        rescheduleStorm()
    end
    function Weather.setStormVar(v)
        Config.WeatherStormVar = math.clamp(v, 0, 30)
        rescheduleStorm()
    end
    function Weather.setMeteorRate(v)
        Config.WeatherMeteorRate = math.clamp(v, 0.25, 3)
        rescheduleMeteors()
    end
    function Weather.setStarRate(v)
        Config.WeatherStarRate = math.clamp(v, 0.25, 3)
        rescheduleStars()
    end
    function Weather.togglePuddles(on)
        Config.WeatherPuddles = on
        puddlesRefresh()
    end
    function Weather.toggleMood(on)
        Config.WeatherMood = on
        if Config.Weather and _curType then
            if on then applyMood(_curType) else clearMood() end
        end
    end
    function Weather.toggleMeteors(on)
        Config.WeatherMeteors = on
        if Config.Weather then if on then startMeteors() else stopMeteors() end end
    end
    function Weather.toggleShootingStars(on)
        Config.WeatherShootingStars = on
        if Config.Weather then if on then startStars() else stopStars() end end
    end
    function Weather.toggleClock(on)
        Config.WeatherClockDial = on
        if Config.Weather then if on then startClock() else stopClock() end end
    end
    function Weather.enableWeather()
        Config.Weather = true
        getFolder(); startFollow()
        applyType(cfg("WeatherType", "Rain"))
        if cfg("WeatherStorm", false) then startStorm() end
        if cfg("WeatherMeteors", false) then startMeteors() end
        if cfg("WeatherShootingStars", false) then startStars() end
        if cfg("WeatherClockDial", false) then startClock() end
    end
    function Weather.disableWeather()
        Config.Weather = false
        stopFollow(); stopStorm(); stopRain(); clearPartLayers(); killFade(); clearMood()
        clearSpecial(); stopMeteors(); stopStars(); stopFireflies(); stopLeafDevil(); stopClock()
        stopPuddles(true)
        if _ambient then pcall(function() _ambient:Destroy() end); _ambient = nil end
        _curType = nil
    end
    local SKY = {
        Space     = { Bk="rbxassetid://159454299", Dn="rbxassetid://159454296", Ft="rbxassetid://159454293", Lf="rbxassetid://159454286", Rt="rbxassetid://159454300", Up="rbxassetid://159454288" },
        Sunset    = { Bk="rbxassetid://264908339", Dn="rbxassetid://264907909", Ft="rbxassetid://264909420", Lf="rbxassetid://264909758", Rt="rbxassetid://264908886", Up="rbxassetid://264907379" },
        Clouds    = { Bk="rbxassetid://570557514", Dn="rbxassetid://570557775", Ft="rbxassetid://570557559", Lf="rbxassetid://570557620", Rt="rbxassetid://570557672", Up="rbxassetid://570557727" },
        Storm     = { Bk="rbxassetid://255027929", Dn="rbxassetid://255027967", Ft="rbxassetid://255027923", Lf="rbxassetid://255027938", Rt="rbxassetid://255027946", Up="rbxassetid://255027960" },
        Winter    = { Bk="rbxassetid://402229526", Dn="rbxassetid://402229596", Ft="rbxassetid://402229293", Lf="rbxassetid://402229368", Rt="rbxassetid://402229417", Up="rbxassetid://402229564" },
        Vaporwave = { Bk="rbxassetid://1417494030", Dn="rbxassetid://1417494146", Ft="rbxassetid://1417494253", Lf="rbxassetid://1417494402", Rt="rbxassetid://1417494499", Up="rbxassetid://1417494643" },
    }
    Weather.SkyboxOrder = { "Off", "Space", "Sunset", "Clouds", "Storm", "Winter", "Vaporwave" }
    local _sky, _skyConn = nil, nil
    local _origSkies = {}
    local function hideMapSkies()
        for _, c in ipairs(Lighting:GetChildren()) do
            if c:IsA("Sky") and not c:GetAttribute("WX_Custom") then
                table.insert(_origSkies, c)
                pcall(function() c.Parent = nil end)
            end
        end
    end
    local function restoreMapSkies()
        for i = #_origSkies, 1, -1 do
            local c = _origSkies[i]
            if c and c.Parent == nil then
                pcall(function() c.Parent = Lighting end)
            end
            _origSkies[i] = nil
        end
    end
    local function buildSky(preset)
        local set = SKY[preset]; if not set then return end
        hideMapSkies()
        local s = Instance.new("Sky")
        s.Name = "_wxSky"; s:SetAttribute("WX_Custom", true)
        s.SkyboxBk, s.SkyboxDn, s.SkyboxFt = set.Bk, set.Dn, set.Ft
        s.SkyboxLf, s.SkyboxRt, s.SkyboxUp = set.Lf, set.Rt, set.Up
        if cfg("SkyboxHideCelestial", false) then
            s.SunAngularSize = 0; s.MoonAngularSize = 0; s.StarCount = 0
            s.CelestialBodiesShown = false
        else
            s.CelestialBodiesShown = true
        end
        s.Parent = Lighting
        _sky = s
    end
    local function startSkyGuard()
        if _skyConn then return end
        _skyConn = Lighting.ChildAdded:Connect(function(c)
            if c:IsA("Sky") and not c:GetAttribute("WX_Custom") and Config.SkyboxPreset and Config.SkyboxPreset ~= "Off" then
                table.insert(_origSkies, c)
                pcall(function() c.Parent = nil end)
            end
        end)
    end
    local function stopSkyGuard()
        if _skyConn then _skyConn:Disconnect(); _skyConn = nil end
    end
    local function clearSky()
        if _sky then pcall(function() _sky:Destroy() end); _sky = nil end
        restoreMapSkies()
    end
    function Weather.setSkybox(preset)
        if preset and not SKY[preset] then preset = "Off" end
        Config.SkyboxPreset = preset
        clearSky()
        if preset == "Off" or preset == nil then stopSkyGuard(); return end
        buildSky(preset); startSkyGuard()
    end
    function Weather.toggleCelestial(hide)
        Config.SkyboxHideCelestial = hide
        if _sky then
            if hide then
                _sky.SunAngularSize = 0; _sky.MoonAngularSize = 0; _sky.StarCount = 0
                _sky.CelestialBodiesShown = false
            else
                _sky.SunAngularSize = 11; _sky.MoonAngularSize = 11; _sky.StarCount = 3000
                _sky.CelestialBodiesShown = true
            end
        end
    end
    local _rainbow = { host = nil, conn = nil }
    local RB_BANDS = {
        { Color3.fromRGB(255, 40, 40),  0.10, 24 },
        { Color3.fromRGB(255, 130, 20), 0.12, 24 },
        { Color3.fromRGB(255, 225, 40), 0.14, 24 },
        { Color3.fromRGB(60, 210, 70),  0.16, 24 },
        { Color3.fromRGB(40, 130, 255), 0.18, 24 },
        { Color3.fromRGB(85, 60, 235),  0.26, 18 },
        { Color3.fromRGB(165, 65, 230), 0.30, 16 },
    }
    local RB_DIST, RB_OY = 430, -20
    local function clearRainbow()
        if _rainbow.conn then _rainbow.conn:Disconnect(); _rainbow.conn = nil end
        if _rainbow.host then pcall(function() _rainbow.host:Destroy() end); _rainbow.host = nil end
    end
    local function buildRainbow()
        local host = Instance.new("Part")
        host.Name = "_wxRainbow"; host.Anchored = true; host.CanCollide = false; host.CanQuery = false
        host.CanTouch = false; host.CastShadow = false; host.Massless = true; host.Transparency = 1
        host.Size = Vec(1, 1, 1); host:SetAttribute("WX_Custom", true)
        local rot = CFrame.Angles(0, 0, math.rad(90))
        local function band(sp, rr, w, col, midT, emit, soft)
            local a0 = Instance.new("Attachment"); a0.CFrame = CFrame.new(-sp, 0, 0) * rot; a0.Parent = host
            local a1 = Instance.new("Attachment"); a1.CFrame = CFrame.new( sp, 0, 0) * rot; a1.Parent = host
            local b = Instance.new("Beam")
            b.Attachment0 = a0; b.Attachment1 = a1; b.Segments = 46; b.FaceCamera = true
            if soft then b.Texture = TX_SOFT; b.TextureMode = Enum.TextureMode.Stretch; b.TextureLength = 1 end
            b.Width0 = w; b.Width1 = w; b.CurveSize0 = rr; b.CurveSize1 = -rr
            b.LightEmission = emit; b.LightInfluence = 0
            b.Color = ColorSequence.new(col)
            local edge = math.min(midT + 0.25, 1)
            b.Transparency = NumberSequence.new({
                NSK(0, 1), NSK(0.08, edge), NSK(0.45, midT), NSK(0.55, midT), NSK(0.92, edge), NSK(1, 1) })
            b.Parent = host
        end
        for i = 1, #RB_BANDS do
            local B = RB_BANDS[i]
            band(340, 320 - (i - 1) * 15, B[3], B[1], B[2], 0.42, false)
        end
        band(340, 165, 52, Color3.fromRGB(242, 248, 255), 0.78, 0.6, true)
        for i = 1, #RB_BANDS do
            band(400, 355 + (i - 1) * 12, 20, RB_BANDS[i][1], 0.64 + i * 0.012, 0.42, false)
        end
        host.Parent = getFolder()
        _rainbow.host = host
    end
    local function startRainbowFollow()
        if _rainbow.conn then return end
        local accum = 0
        _rainbow.conn = RunService.Heartbeat:Connect(function(dt)
            if not cfg("WeatherRainbow", false) then return end
            accum = accum + dt; if accum < 0.08 then return end
            accum = 0
            local cam = Camera; if not cam then return end
            local host = _rainbow.host; if not host or not host.Parent then return end
            local p = cam.CFrame.Position
            local hp = Vec(p.X, p.Y + RB_OY, p.Z - RB_DIST)
            host.CFrame = CFrame.lookAt(hp, Vec(p.X, hp.Y, p.Z))
        end)
    end
    function Weather.toggleRainbow(on)
        Config.WeatherRainbow = on
        clearRainbow()
        if not on then return end
        buildRainbow(); startRainbowFollow()
    end
    function Weather.toggleGodRays(on)
        Config.WeatherGodRays = on
        if _rays then pcall(function() _rays:Destroy() end); _rays = nil end
        if not on then return end
        local r = Instance.new("SunRaysEffect")
        r.Name = "_wxRays"; r:SetAttribute("WX_Custom", true)
        local I = iAmt()
        r.Intensity = 0.10 + 0.18 * I
        r.Spread = 1.1 - 0.2 * I
        r.Parent = Lighting
        _rays = r
    end
    function Weather.init()
        if Config.Weather then pcall(Weather.enableWeather) end
        if Config.SkyboxPreset and Config.SkyboxPreset ~= "Off" then pcall(Weather.setSkybox, Config.SkyboxPreset) end
        if Config.WeatherGodRays then pcall(Weather.toggleGodRays, true) end
        if Config.WeatherRainbow then pcall(Weather.toggleRainbow, true) end
    end
    function Weather.unload()
        pcall(Weather.disableWeather)
        stopSkyGuard(); clearSky(); clearRainbow()
        if _rays then pcall(function() _rays:Destroy() end); _rays = nil end
        if _folder then pcall(function() _folder:Destroy() end); _folder = nil end
        _lightFolder = nil
    end
end)()


-- Wrappers para shako UI
function StartWeather()
    config.WeatherEnabled = true
    config.Weather = true
    pcall(function() if Config then Config.Weather = true end end)
    -- rebuild completo (map load destruye _wx)
    pcall(function()
        if Weather and Weather.disableWeather then Weather.disableWeather() end
    end)
    task.defer(function()
        pcall(function() Weather.enableWeather() end)
        if config.SkyboxPreset and config.SkyboxPreset ~= "Off" then
            pcall(function() if Weather and Weather.setSkybox then Weather.setSkybox(config.SkyboxPreset) end end)
        end
        if skyboxData and skyboxData.enabled then pcall(ApplySkybox) end
    end)
    if not getgenv()._ShakoQuiet then Notify("Weather", "ON · " .. tostring(config.WeatherType or "Rain")) end
end

-- ==================== THIRD PERSON (LuaHook style) ====================
local _tpRP = nil
local _tpConn = nil
function StartThirdPerson()
    config.ThirdPerson = true
    config.ThirdPersonEnabled = true
    if _tpConn then return end
    _tpConn = RunService.RenderStepped:Connect(function()
        if not (config.ThirdPerson or config.ThirdPersonEnabled) then return end
        if config.RagebotEnabled then return end
        local cam = workspace.CurrentCamera
        if not cam then return end
        local char = LocalPlayer.Character
        if not char then return end
        local dist = math.clamp(tonumber(config.ThirdPersonDistance) or 12, 4, 30)
        local c = cam.CFrame
        local desiredPos = c.Position - c.LookVector * dist
        if _tpRP == nil then
            _tpRP = RaycastParams.new()
            _tpRP.FilterType = Enum.RaycastFilterType.Exclude
        end
        _tpRP.FilterDescendantsInstances = { char }
        local hit = workspace:Raycast(c.Position, -c.LookVector * dist, _tpRP)
        if hit then desiredPos = hit.Position + c.LookVector * 0.5 end
        cam.CFrame = (c - c.Position) + desiredPos
        -- show local body
        for _, p in ipairs(char:GetDescendants()) do
            if p:IsA("BasePart") and p.Name ~= "HumanoidRootPart" then
                if p.LocalTransparencyModifier > 0.5 then
                    p.LocalTransparencyModifier = 0
                end
            end
        end
    end)
    if not getgenv()._ShakoQuiet then Notify("Camera", "Third person ON") end
end
function StopThirdPerson()
    config.ThirdPerson = false
    config.ThirdPersonEnabled = false
    if _tpConn then pcall(function() _tpConn:Disconnect() end) _tpConn = nil end
    if not getgenv()._ShakoQuiet and not (getgenv()._ShakoBootAt and tick() - getgenv()._ShakoBootAt < 8) then
        Notify("Camera", "Third person OFF")
    end
end


function StopWeather()
    config.WeatherEnabled = false
    Config.Weather = false
    pcall(function() Weather.disableWeather() end)
    if not getgenv()._ShakoQuiet and not (getgenv()._ShakoBootAt and tick() - getgenv()._ShakoBootAt < 8) then
        Notify("Weather", "OFF")
    end
end

-- ==================== EMOTES (siempre loop + keep playing) ====================
EMOTES = {
    -- IDs públicos que suelen cargar en cliente
    ["Floss"] = "5917459365",
    ["Orange Justice"] = "3934987096",
    ["Take The L"] = "3333499508",
    ["Default Dance"] = "4265725525",
    ["Electro Shuffle"] = "3489178132",
    ["Fresh"] = "3183986796",
    ["Robot"] = "4555808220",
    ["Hype"] = "3360689775",
    ["Dab"] = "3333499505",
    ["Laugh"] = "3333430984",
    ["Wave"] = "507770239",
    ["Point"] = "507770453",
    ["Cheer"] = "507770677",
    ["Gangnam"] = "5918726674",
    ["Shuffle"] = "434870490",
    ["Breakdance"] = "5918726674",
    ["Twist"] = "3333492741",
    ["Zombie"] = "3489174152",
    -- TUS IDs (cargados vía GetObjects / InsertService)
    ["Custom A"] = "86329197824055",
    ["Custom B"] = "91729309021707",
    ["Custom C"] = "133811691098518",
    ["Custom D"] = "133375187065498",
    -- nuevas
    ["Custom E"] = "130601328561804",
    ["Custom F"] = "129571341119376",
    ["Custom G"] = "79902725961872",
    ["Custom H"] = "119060168059843",
    ["Custom I"] = "82995540773684",
    ["Custom J"] = "78224683906191",
    ["Custom K"] = "73562814360939",
    ["Custom L"] = "88425531063616",
    -- aliases por id
    ["86329197824055"] = "86329197824055",
    ["91729309021707"] = "91729309021707",
    ["133811691098518"] = "133811691098518",
    ["133375187065498"] = "133375187065498",
    ["130601328561804"] = "130601328561804",
    ["129571341119376"] = "129571341119376",
    ["79902725961872"] = "79902725961872",
    ["119060168059843"] = "119060168059843",
    ["82995540773684"] = "82995540773684",
    ["78224683906191"] = "78224683906191",
    ["73562814360939"] = "73562814360939",
    ["88425531063616"] = "88425531063616",
}

local currentAnimTrack = nil
local emoteKeepConn = nil
local lastEmoteId = nil
local emoteLastRestart = 0

function StopEmote()
    if emoteKeepConn then
        pcall(function() emoteKeepConn:Disconnect() end)
        emoteKeepConn = nil
    end
    if currentAnimTrack then
        pcall(function() currentAnimTrack:Stop(0.1) end)
        currentAnimTrack = nil
    end
    config.EmoteEnabled = false
end

local function getAnimator(hum)
    if not hum then return nil end
    local animator = hum:FindFirstChildOfClass("Animator")
    if not animator then
        animator = Instance.new("Animator")
        animator.Parent = hum
    end
    return animator
end

-- Cache de Animation ya resueltas por id
local _animCache = {}

local function resolveAnimationObject(id)
    id = tostring(id or ""):gsub("%D", "")
    if id == "" then return nil end
    if _animCache[id] and _animCache[id].Parent ~= nil then
        return _animCache[id]
    end

    local function fromInstance(inst)
        if not inst then return nil end
        if inst:IsA("Animation") then return inst end
        if inst:IsA("KeyframeSequence") then
            -- convertir via AnimationClipProvider si existe
            local ok, animId = pcall(function()
                local ACP = game:GetService("AnimationClipProvider")
                return ACP:RegisterAnimationClip(inst)
            end)
            if ok and animId then
                local a = Instance.new("Animation")
                a.AnimationId = animId
                return a
            end
        end
        -- buscar Animation adentro (asset de catálogo / package)
        for _, d in ipairs(inst:GetDescendants()) do
            if d:IsA("Animation") then return d end
        end
        if inst:IsA("Model") or inst:IsA("Folder") then
            local a = inst:FindFirstChildWhichIsA("Animation", true)
            if a then return a end
        end
        return nil
    end

    -- 1) game:GetObjects (exploit) — el más fiable para IDs de catálogo
    local ok, objs = pcall(function()
        return game:GetObjects("rbxassetid://" .. id)
    end)
    if ok and type(objs) == "table" then
        for _, o in ipairs(objs) do
            local a = fromInstance(o)
            if a then
                _animCache[id] = a
                return a
            end
        end
    end

    -- 2) InsertService:LoadAsset
    pcall(function()
        local IS = game:GetService("InsertService")
        local model = IS:LoadAsset(tonumber(id))
        if model then
            local a = fromInstance(model)
            if a then
                _animCache[id] = a
                model.Parent = nil
                return a
            end
            pcall(function() model:Destroy() end)
        end
    end)
    if _animCache[id] then return _animCache[id] end

    -- 3) Animation "manual" con varios formatos de URL
    for _, uri in ipairs({
        "rbxassetid://" .. id,
        "http://www.roblox.com/asset/?id=" .. id,
        "https://www.roblox.com/asset/?id=" .. id,
    }) do
        local a = Instance.new("Animation")
        a.Name = "ShakoEmote_" .. id
        a.AnimationId = uri
        _animCache[id] = a
        return a
    end
    return nil
end

local function loadTrack(animator, id)
    id = tostring(id or ""):gsub("%D", "")
    if id == "" or not animator then return nil end

    local animObj = resolveAnimationObject(id)
    if animObj and animObj:IsA("Animation") then
        local ok, track = pcall(function() return animator:LoadAnimation(animObj) end)
        if ok and track then return track end
        local hum = animator.Parent
        if hum and hum:IsA("Humanoid") then
            ok, track = pcall(function() return hum:LoadAnimation(animObj) end)
            if ok and track then return track end
        end
    end

    -- último recurso: Animation nueva directa
    for _, uri in ipairs({
        "rbxassetid://" .. id,
        "http://www.roblox.com/asset/?id=" .. id,
    }) do
        local anim = Instance.new("Animation")
        anim.AnimationId = uri
        local ok, track = pcall(function() return animator:LoadAnimation(anim) end)
        if ok and track then return track end
        local hum = animator.Parent
        if hum and hum:IsA("Humanoid") then
            ok, track = pcall(function() return hum:LoadAnimation(anim) end)
            if ok and track then return track end
        end
    end
    return nil
end

local function stopCompeting(animator, keep)
    pcall(function()
        for _, t in ipairs(animator:GetPlayingAnimationTracks()) do
            if t ~= keep then
                local pr = t.Priority
                -- no matar Core de muerte; sí Movement/Action bajos
                if pr == Enum.AnimationPriority.Movement
                    or pr == Enum.AnimationPriority.Idle
                    or pr == Enum.AnimationPriority.Action
                    or pr == Enum.AnimationPriority.Action2
                    or pr == Enum.AnimationPriority.Action3 then
                    pcall(function() t:Stop(0) end)
                end
            end
        end
    end)
end

function PlayEmote(animId, speed)
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local animator = getAnimator(hum)
    if not animator then return end

    local id = tostring(animId or config.EmoteId or ""):gsub("%D", "")
    if id == "" and config.EmoteName and EMOTES[config.EmoteName] then
        id = tostring(EMOTES[config.EmoteName]):gsub("%D", "")
    end
    if id == "" then id = "5917459365" end

    lastEmoteId = id
    config.EmoteId = id
    config.EmoteEnabled = true
    config.EmoteLoop = true
    config.EmoteKeepPlaying = true

    if currentAnimTrack then
        pcall(function() currentAnimTrack:Stop(0) end)
        currentAnimTrack = nil
    end

    local track = loadTrack(animator, id)
    if not track then
        -- reintento lento (GetObjects a veces tarda)
        task.wait(0.15)
        track = loadTrack(animator, id)
    end
    if not track then
        Notify("Emote", "ID no cargó: " .. id .. " (¿privado/inválido?)")
        return
    end
    Notify("Emote", "ON · " .. id)

    pcall(function()
        track.Priority = Enum.AnimationPriority.Action4
        track.Looped = true
        track:Play(0.05, 1, tonumber(speed) or tonumber(config.EmoteSpeed) or 1)
        track:AdjustWeight(1, 0)
    end)
    currentAnimTrack = track
    stopCompeting(animator, track)

    -- loop keep-alive (Rivals pisa anims al moverse)
    if emoteKeepConn then
        pcall(function() emoteKeepConn:Disconnect() end)
        emoteKeepConn = nil
    end
    emoteKeepConn = RunService.Heartbeat:Connect(function()
        if not config.EmoteEnabled then return end
        local now = tick()
        if now - emoteLastRestart < 0.12 then return end -- throttle
        local c = LocalPlayer.Character
        local h = c and c:FindFirstChildOfClass("Humanoid")
        if not h or h.Health <= 0 then return end
        local a = getAnimator(h)
        if not a then return end
        local t = currentAnimTrack
        local need = (not t) or (t.IsPlaying ~= true)
        -- también si el weight bajó mucho
        if t and t.IsPlaying then
            local w = 1
            pcall(function() w = t.WeightTarget or 1 end)
            if w < 0.3 then need = true end
        end
        if need then
            emoteLastRestart = now
            local nt = loadTrack(a, lastEmoteId or config.EmoteId or "5917459365")
            if nt then
                pcall(function()
                    nt.Priority = Enum.AnimationPriority.Action4
                    nt.Looped = true
                    nt:Play(0, 1, tonumber(config.EmoteSpeed) or 1)
                    nt:AdjustWeight(1, 0)
                end)
                currentAnimTrack = nt
                stopCompeting(a, nt)
            end
        else
            pcall(function()
                t.Priority = Enum.AnimationPriority.Action4
                t.Looped = true
                local spd = tonumber(config.EmoteSpeed) or 1
                t:AdjustSpeed(spd)
                t:AdjustWeight(1, 0)
                -- forzar speed si el juego lo resetea
                if math.abs((t.Speed or 1) - spd) > 0.05 then
                    t:Play(0, 1, spd)
                end
            end)
            if now - emoteLastRestart > 0.5 then
                emoteLastRestart = now
                stopCompeting(a, t)
            end
        end
    end)

    -- emote remoto del juego (si existe)
    pcall(function()
        local remote = game:GetService("ReplicatedStorage").Remotes.Replication.Fighter:FindFirstChild("UseEmoteByName")
        if remote and config.EmoteName and config.EmoteName ~= "Custom" then
            remote:FireServer(tostring(config.EmoteName))
        end
    end)
end

-- auto resume al respawn si estaba activo
pcall(function()
    LocalPlayer.CharacterAdded:Connect(function()
        if not config.EmoteEnabled then return end
        task.delay(0.6, function()
            if config.EmoteEnabled then
                PlayEmote(lastEmoteId or config.EmoteId, config.EmoteSpeed)
            end
        end)
    end)
end)

local _visAcc = 0

-- ==================== SHAKO SPINNING CROSSHAIR (logo purple) ====================
local ShakoCross = {
    gui = nil,
    frame = nil,
    lines = {},
    text = nil,
    conn = nil,
    rotation = 0,
    hueT = 0,
}

local function shakoPurpleColor(t)
    -- colores del logo: blanco plateado <-> morado neon
    local pulse = (math.sin(t * 2.2) + 1) * 0.5
    local deep = Color3.fromRGB(120, 40, 220)
    local mid  = Color3.fromRGB(168, 85, 255)
    local bright = Color3.fromRGB(210, 140, 255)
    local white = Color3.fromRGB(255, 255, 255)
    if pulse < 0.5 then
        return deep:Lerp(mid, pulse * 2)
    else
        return mid:Lerp(bright:Lerp(white, 0.35), (pulse - 0.5) * 2)
    end
end

function ShakoCross.destroy()
    if ShakoCross.conn then pcall(function() ShakoCross.conn:Disconnect() end) ShakoCross.conn = nil end
    if ShakoCross.gui then pcall(function() ShakoCross.gui:Destroy() end) end
    ShakoCross.gui, ShakoCross.frame, ShakoCross.text = nil, nil, nil
    ShakoCross.lines = {}
end

function ShakoCross.build()
    ShakoCross.destroy()
    local pg = LocalPlayer:FindFirstChild("PlayerGui") or LocalPlayer:WaitForChild("PlayerGui", 5)
    if not pg then return end
    local gui = Instance.new("ScreenGui")
    gui.Name = "ShakoSpinningCrosshair"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.DisplayOrder = 50
    gui.Parent = pg
    ShakoCross.gui = gui

    local ox = tonumber(config.CrosshairOffsetX) or 0
    local oy = tonumber(config.CrosshairOffsetY) or 0
    local crosshair = Instance.new("Frame")
    crosshair.Name = "Crosshair"
    crosshair.Size = UDim2.fromOffset(200, 200)
    crosshair.Position = UDim2.new(0.5, ox, 0.5, oy)
    crosshair.AnchorPoint = Vector2.new(0.5, 0.5)
    crosshair.BackgroundTransparency = 1
    crosshair.Parent = gui
    ShakoCross.frame = crosshair

    local lineLength = tonumber(config.CrosshairSize) or 25
    local lineThickness = tonumber(config.CrosshairThickness) or 2
    local outlineThickness = config.CrosshairOutline and 0.3 or 0
    local lineGap = tonumber(config.CrosshairGap) or 10
    local halfGap = lineGap / 2
    local col0 = Color3.fromRGB(168, 85, 255)

    local function createLine(size, position, anchorPoint)
        local line = Instance.new("Frame")
        line.Size = size
        line.Position = position
        line.AnchorPoint = anchorPoint
        line.BackgroundColor3 = col0
        line.BorderSizePixel = 0
        line.Parent = crosshair
        if outlineThickness > 0 then
            local stroke = Instance.new("UIStroke")
            stroke.Color = Color3.fromRGB(0, 0, 0)
            stroke.Thickness = outlineThickness
            stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual
            stroke.Parent = line
        end
        table.insert(ShakoCross.lines, line)
    end

    createLine(UDim2.fromOffset(lineThickness, lineLength), UDim2.new(0.5, 0, 0.5, -halfGap), Vector2.new(0.5, 1))
    createLine(UDim2.fromOffset(lineThickness, lineLength), UDim2.new(0.5, 0, 0.5, halfGap), Vector2.new(0.5, 0))
    createLine(UDim2.fromOffset(lineLength, lineThickness), UDim2.new(0.5, -halfGap, 0.5, 0), Vector2.new(1, 0.5))
    createLine(UDim2.fromOffset(lineLength, lineThickness), UDim2.new(0.5, halfGap, 0.5, 0), Vector2.new(0, 0.5))

    local textOff = tonumber(config.CrosshairTextOffsetY) or 28
    local hollow = Instance.new("TextLabel")
    hollow.Name = "ShakoText"
    hollow.Size = UDim2.fromOffset(220, 35)
    hollow.Position = UDim2.new(0.5, ox, 0.5, oy + textOff)
    hollow.AnchorPoint = Vector2.new(0.5, 0)
    hollow.BackgroundTransparency = 1
    hollow.Text = tostring(config.CrosshairText or "")
    hollow.TextColor3 = Color3.fromRGB(255, 255, 255)
    hollow.TextSize = tonumber(config.CrosshairTextSize) or 22
    hollow.Font = Enum.Font.GothamBold
    hollow.TextStrokeColor3 = Color3.fromRGB(40, 0, 80)
    hollow.TextStrokeTransparency = 0.2
    hollow.TextXAlignment = Enum.TextXAlignment.Center
    hollow.Parent = gui
    ShakoCross.text = hollow

    ShakoCross.rotation = 0
    ShakoCross.hueT = 0
    ShakoCross.conn = RunService.RenderStepped:Connect(function(dt)
        if not config.Crosshair then
            if ShakoCross.gui then ShakoCross.gui.Enabled = false end
            return
        end
        if ShakoCross.gui then ShakoCross.gui.Enabled = true end
        local speed = tonumber(config.CrosshairSpinSpeed) or 120
        local ox = tonumber(config.CrosshairOffsetX) or 0
        local oy = tonumber(config.CrosshairOffsetY) or 0
        if ShakoCross.frame then
            ShakoCross.frame.Position = UDim2.new(0.5, ox, 0.5, oy)
        end
        if ShakoCross.text then
            local textOff = tonumber(config.CrosshairTextOffsetY) or 28
            ShakoCross.text.Position = UDim2.new(0.5, ox, 0.5, oy + textOff)
        end
        if config.CrosshairSpin ~= false then
            ShakoCross.rotation = ShakoCross.rotation + speed * (dt or 0.016)
            if ShakoCross.frame then ShakoCross.frame.Rotation = ShakoCross.rotation end
        else
            if ShakoCross.frame then ShakoCross.frame.Rotation = 0 end
        end
        ShakoCross.hueT = ShakoCross.hueT + (dt or 0.016)
        local col = shakoPurpleColor(ShakoCross.hueT)
        if not config.CrosshairRainbow then
            col = Color3.fromRGB(168, 85, 255)
        end
        for _, line in ipairs(ShakoCross.lines) do
            line.BackgroundColor3 = col
        end
        if ShakoCross.text then
            ShakoCross.text.TextColor3 = col
            ShakoCross.text.Text = tostring(config.CrosshairText or "")
            ShakoCross.text.TextSize = tonumber(config.CrosshairTextSize) or 22
        end
    end)
end

function StartCrosshair()
    config.Crosshair = true
    ShakoCross.build()
end
function StopCrosshair()
    config.Crosshair = false
    ShakoCross.destroy()
end


-- OPT: EnsureCameraFree cada 0.5s (no RenderStepped)
task.spawn(function()
    while true do
        task.wait(0.5)
        pcall(EnsureCameraFree)
    end
end)

function HookCharacter(char)
    task.defer(ResetCameraNormal)
    local hum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 5)
    if hum then
        hum.Died:Connect(function()
            UECounterReady = false
            if config.AutoRespawn then
                task.delay(0.3, function() pcall(function() LocalPlayer:LoadCharacter() end) end)
            end
        end)
    end
end

UserInputService.JumpRequest:Connect(function()
    local hum = getHum()
    local hrp = getHRP()
    if not hum then return end

    if config.InfinityJump then
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Jumping) end)
    end

    -- Jump power forzado (Rivals ignora JumpPower a menudo)
    if config.JumpPowerEnabled and config.JumpBoostForce ~= false and hrp then
        local jp = config.JumpPower or 100
        pcall(function()
            hum.UseJumpPower = true
            hum.JumpPower = jp
            if hum.JumpHeight ~= nil then hum.JumpHeight = math.clamp(jp / 20, 2, 500) end
        end)
        -- impulso vertical directo
        local vel = hrp.AssemblyLinearVelocity
        local boostY = math.max(jp * 0.45, 40)
        hrp.AssemblyLinearVelocity = Vector3.new(vel.X, boostY, vel.Z)
    end
end)

-- Rivals pisa WalkSpeed/JumpPower: re-aplicar al cambiar
task.spawn(function()
    local lastHum
    while true do
        task.wait(0.15)
        local hum = getHum()
        if hum and hum ~= lastHum then
            lastHum = hum
            pcall(function()
                hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
                    if config.SpeedEnabled then
                        pcall(function() hum.WalkSpeed = config.WalkSpeed or 50 end)
                    end
                    -- si OFF no tocamos (deja el valor del juego)
                end)
                hum:GetPropertyChangedSignal("JumpPower"):Connect(function()
                    if config.JumpPowerEnabled then
                        pcall(function()
                            hum.UseJumpPower = true
                            hum.JumpPower = config.JumpPower or 100
                        end)
                    end
                end)
            end)
        end
        -- guardar defaults del juego
        if hum then
            pcall(function()
                if not config._savedDefaultWS then
                    config.DefaultWalkSpeed = hum.WalkSpeed
                    config._savedDefaultWS = true
                end
                if not config._savedDefaultJP then
                    config.DefaultJumpPower = hum.JumpPower
                    if hum.JumpHeight then config.DefaultJumpHeight = hum.JumpHeight end
                    config._savedDefaultJP = true
                end
            end)
        end
        -- SOLO aplicar si el usuario tiene toggle ON (nunca auto)
        if config.SpeedEnabled then ApplySpeedState() end
        if config.JumpPowerEnabled then ApplyJumpState() end
    end
end)

-- Config system (métodos en tabla = 1 register en vez de 10+)
function Cfg.EnsureFolder()
    if not isfolder then return false end
    if not isfolder(Cfg.folder) then pcall(function() makefolder(Cfg.folder) end) end
    return isfolder and isfolder(Cfg.folder)
end
function Cfg.Sanitize(n)
    if not n or n == "" then return nil end
    n = tostring(n):gsub("[^%w%_% %-]", ""):gsub("%s+", "_")
    return n ~= "" and n or nil
end
function Cfg.List()
    local list = {"(ninguna)"}
    if not Cfg.EnsureFolder() then return list end
    local ok, files = pcall(function() return listfiles(Cfg.folder) end)
    if ok and files then
        for _, path in ipairs(files) do
            local name = tostring(path):match("([^/\\]+)%.json$")
            if name then table.insert(list, name) end
        end
    end
    return list
end

function Cfg.Save(name, overwrite)
    name = Cfg.Sanitize(name)
    if not name or not Cfg.EnsureFolder() then
        Notify("Config", "No se pudo guardar (folder/name)")
        return false
    end
    local path = Cfg.folder .. "/" .. name .. ".json"
    if isfile and isfile(path) and not overwrite then
        Notify("Config", "Ya existe — usa Overwrite")
        return false
    end

    -- sincronizar skyboxData → config antes de guardar
    pcall(function()
        if skyboxData then
            config.SkyboxEnabled = skyboxData.enabled and true or false
            config.SkyboxName = tostring(skyboxData.selected or "None")
            config.SkyRotate = skyboxData.auto_rotate and true or false
            if skyboxData.enabled and skyboxData.selected and skyboxData.selected ~= "None" then
                if not config.SkyboxPreset or config.SkyboxPreset == "Off" then
                    -- no pisar preset de Weather si ya hay
                end
            end
        end
        config.Weather = config.WeatherEnabled and true or false
    end)
    local t = {}
    -- todo lo serializable de config
    for k, v in pairs(config) do
        local ty = typeof(v)
        if ty == "number" or ty == "string" or ty == "boolean" then
            t[k] = v
        end
    end

    -- Spoof / rank / device
    pcall(function()
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        local rs = RivalsSpoof or st or {}
        t._RS = {
            avatar_userid = tonumber(rs.avatar_userid) or tonumber(config.SkinTargetUserId) or 0,
            spoof_avatar = rs.spoof_avatar and true or false,
            display_name = rs.display_name and true or false,
            display_name_value = tostring(rs.display_name_value or ""),
            username = rs.username and true or false,
            username_value = tostring(rs.username_value or ""),
            device = tostring(rs.device or "Desktop"),
            spoof_device = rs.spoof_device and true or false,
            badges = {},
            charm = {},
            leaderboard = {},
        }
        if type(rs.badges) == "table" then
            for bk, bv in pairs(rs.badges) do t._RS.badges[bk] = bv and true or false end
        end
        if type(rs.charm) == "table" then
            t._RS.charm = {
                charm_rank = tostring(rs.charm.charm_rank or "Archnemesis"),
                arch_rank = tonumber(rs.charm.arch_rank) or 1,
                s0_charm = rs.charm.s0_charm and true or false,
                s1_charm = rs.charm.s1_charm and true or false,
                s2_charm = rs.charm.s2_charm and true or false,
                s3_charm = rs.charm.s3_charm and true or false,
            }
        end
        if type(rs.leaderboard) == "table" then
            local lb = rs.leaderboard
            t._RS.leaderboard = {
                ELO = lb.ELO and true or false, elo_value = tonumber(lb.elo_value) or 0,
                Streak = lb.Streak and true or false, streak_value = tonumber(lb.streak_value) or 0,
                Kills = lb.Kills and true or false, kills_value = tonumber(lb.kills_value) or 0,
                Wins = lb.Wins and true or false, wins_value = tonumber(lb.wins_value) or 0,
                Level = lb.Level and true or false, level_value = tonumber(lb.level_value) or 0,
            }
        elseif type(rs.lb) == "table" then
            local lb = rs.lb
            t._RS.leaderboard = {
                ELO = lb.elo and true or false, elo_value = tonumber(lb.elo_value) or 0,
                Streak = lb.streak and true or false, streak_value = tonumber(lb.streak_value) or 0,
                Kills = lb.kills and true or false, kills_value = tonumber(lb.kills_value) or 0,
                Wins = lb.wins and true or false, wins_value = tonumber(lb.wins_value) or 0,
                Level = lb.level and true or false, level_value = tonumber(lb.level_value) or 0,
            }
        end
    end)

    -- Unlock All skins / weapons (AURORA skinchanger_state)
    pcall(function()
        local st = getgenv()._ShakoStates
        if not st then return end
        t._Unlock = {
            fake_owned = {},
            fake_weapon_owned = {},
            equipped = {},
        }
        if st.skinchanger_state then
            if type(st.skinchanger_state.fake_owned) == "table" then
                for k, v in pairs(st.skinchanger_state.fake_owned) do
                    if v then t._Unlock.fake_owned[tostring(k)] = true end
                end
            end
            if type(st.skinchanger_state.fake_weapon_owned) == "table" then
                -- puede ser array o map
                for k, v in pairs(st.skinchanger_state.fake_weapon_owned) do
                    if type(v) == "table" and v.Name then
                        table.insert(t._Unlock.fake_weapon_owned, v.Name)
                    elseif type(v) == "string" then
                        table.insert(t._Unlock.fake_weapon_owned, v)
                    elseif v == true then
                        table.insert(t._Unlock.fake_weapon_owned, tostring(k))
                    end
                end
            end
            if type(st.skinchanger_state.equipped) == "table" then
                for k, v in pairs(st.skinchanger_state.equipped) do
                    t._Unlock.equipped[tostring(k)] = v
                end
            end
        end
        if st.inventory_state and type(st.inventory_state.fake_owned) == "table" then
            t._Unlock.unclaimed = st.inventory_state.fake_owned
        end
    end)

    -- snapshot UI flags (toggles + options) para restaurar visual al load
    pcall(function()
        local Toggles = (Library and Library.Toggles) or rawget(getgenv(), "Toggles")
        local Options = (Library and Library.Options) or rawget(getgenv(), "Options")
        t._UI = { toggles = {}, options = {} }
        if type(Toggles) == "table" then
            for flag, toggle in pairs(Toggles) do
                local v = nil
                pcall(function()
                    if toggle.Value ~= nil then v = toggle.Value
                    elseif toggle.Get then v = toggle:Get()
                    elseif toggle.GetValue then v = toggle:GetValue()
                    end
                end)
                if typeof(v) == "boolean" then
                    t._UI.toggles[tostring(flag)] = v
                end
            end
        end
        if type(Options) == "table" then
            for flag, opt in pairs(Options) do
                local v = nil
                pcall(function()
                    if opt.Value ~= nil then v = opt.Value
                    elseif opt.Get then v = opt:Get()
                    elseif opt.GetValue then v = opt:GetValue()
                    end
                end)
                local ty = typeof(v)
                if ty == "number" or ty == "string" or ty == "boolean" then
                    t._UI.options[tostring(flag)] = v
                end
            end
        end
        -- forzar avatar id en options y config
        local aid = tonumber(config.SkinTargetUserId)
        pcall(function()
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            local rs = RivalsSpoof or st
            if rs and tonumber(rs.avatar_userid) then aid = tonumber(rs.avatar_userid) end
        end)
        if aid then
            t.SkinTargetUserId = aid
            if t._RS then t._RS.avatar_userid = aid end
            if t._UI and t._UI.options then
                t._UI.options["VictimUserId"] = tostring(aid)
                t._UI.options["SkinTargetUserId"] = tostring(aid)
                t._UI.options["AvatarUserId"] = tostring(aid)
            end
        end
    end)

    local ok, enc = pcall(function() return HttpService:JSONEncode(t) end)
    if not ok or not enc then
        Notify("Config", "JSON encode fail")
        return false
    end
    local wok = pcall(function() writefile(path, enc) end)
    if wok then
        Notify("Config", "Guardada: " .. name)
    else
        Notify("Config", "writefile FAIL")
    end
    return wok
end

function Cfg.SyncUIFromConfig(data)
    -- Restaura visual de toggles/sliders/inputs según config + data._UI
    data = data or {}
    -- Obsidian Toggle:SetValue SIEMPRE dispara Callback → desactiva/activa features.
    -- Durante sync solo actualizamos Value + Display SIN callback.
    local function setToggle(toggle, val)
        if not toggle or typeof(val) ~= "boolean" then return end
        pcall(function()
            local oldCb = toggle.Callback
            local oldChanged = toggle.Changed
            local oldRun = toggle.RunChanged
            toggle.Callback = function() end
            if type(oldChanged) == "table" and oldChanged.Clear then
                -- leave signal
            end
            pcall(function()
                if toggle.RunChanged then
                    toggle.RunChanged = function() end
                end
            end)
            pcall(function()
                toggle.Value = val
                if toggle.Display then toggle:Display() end
            end)
            -- SetValue con callbacks anulados
            pcall(function()
                if toggle.SetValue then toggle:SetValue(val) end
            end)
            toggle.Callback = oldCb
            pcall(function()
                if oldRun then toggle.RunChanged = oldRun end
            end)
            toggle.Value = val
            pcall(function() if toggle.Display then toggle:Display() end end)
        end)
    end
    local function setOption(opt, val)
        if not opt or val == nil then return end
        pcall(function()
            local oldCb = opt.Callback
            opt.Callback = function() end
            pcall(function()
                if opt.SetValue then opt:SetValue(val) end
            end)
            pcall(function() opt.Value = val end)
            if typeof(val) == "string" then
                pcall(function() if opt.SetText then opt:SetText(val) end end)
                pcall(function()
                    -- Input visual box
                    if opt.Box and opt.Box.Text ~= nil then opt.Box.Text = tostring(val) end
                end)
            elseif typeof(val) == "number" then
                pcall(function() if opt.Display then opt:Display() end end)
                pcall(function() if opt.SetValue then opt:SetValue(val) end end)
            end
            opt.Callback = oldCb
            opt.Value = val
            pcall(function() if opt.Display then opt:Display() end end)
        end)
    end

    pcall(function()
        local Toggles = (Library and Library.Toggles) or rawget(getgenv(), "Toggles")
        local Options = (Library and Library.Options) or rawget(getgenv(), "Options")

        local flagMap = {
            EnableHVH = "Enabled",
            RagebotToggle = "RagebotEnabled",
            VoidSpam = "VoidSpamEnabled",
            FlyToggle = "FlyEnabled",
            NoclipToggle = "NoclipEnabled",
            SpeedToggle = "SpeedEnabled",
            JumpPowerToggle = "JumpPowerEnabled",
            SlideBoost = "SlideBoostEnabled",
            ESPEnabled = "ESPEnabled",
            AimbotToggle = "AimbotEnabled",
            SilentToggle = "SilentAimEnabled",
            SilentAimEnabled = "SilentAimEnabled",
            Triggerbot = "TriggerbotEnabled",
            PredDodge = "PredDodgeEnabled",
            StretchedRes = "StretchedRes",
            UECounterToggle = "UECounterEnabled",
            AntiUnnamed = "UECounterEnabled",
            RapidFire = "RapidFire",
            FastMelee = "FastMelee",
            NoRecoilToggle = "NoRecoil",
            SoftStick = "SoftStick",
            Strafe = "StrafeEnabled",
            Orbit = "OrbitEnabled",
            TargetLock = "TargetLock",
            TeamCheckMain = "TeamCheck",
            Hitbox = "HitboxExpand",
            AntiAim = "AntiAimEnabled",
            AntiFling = "AntiFling",
            InfinityJump = "InfinityJump",
            FakePosition = "FakePosition",
            DesyncJitter = "DesyncJitter",
            StickMultiPos = "StickMultiPos",
            StickDesync = "StickDesync",
            StickFireHead = "StickFireHead",
            CameraIndependent = "CameraIndependent",
            HitSoundEnabled = "HitSoundEnabled",
            KillSoundEnabled = "KillSoundEnabled",
            AntiKick = "AntiKick",
            AutoRespawn = "AutoRespawn",
            Fullbright = "Fullbright",
            WeatherEnabled = "WeatherEnabled",
            VoidEvade = "VoidEvade",
            SkinchangerEnabled = "SkinchangerEnabled",
            ESPGlow = "ESPGlow",
            ESPTracers = "ESPTracers",
            ESPSkeleton = "ESPSkeleton",
            ESPShowBox = "ESPShowBox",
            ESPShowName = "ESPShowName",
            ESPShowDist = "ESPShowDist",
            ESPShowHP = "ESPShowHP",
            ESPTeamCheck = "ESPTeamCheck",
            RageDesync = "RageDesync",
            RageUseItem = "RageUseItem",
            RagebotInfiniteRange = "RagebotInfiniteRange",
            RagebotTeamCheck = "RagebotTeamCheck",
            RivalsSpoofDisp = "display_name", -- special RS
            RivalsSpoofUser = "username",
            SpoofAvatar = "spoof_avatar",
            FakeRobux = "fake_robux",
        }

        -- 1) Prefer snapshot _UI.toggles (estado exacto de la UI al guardar)
        local uiT = type(data._UI) == "table" and data._UI.toggles or nil
        if type(Toggles) == "table" then
            for flag, toggle in pairs(Toggles) do
                local val = nil
                if uiT and uiT[tostring(flag)] ~= nil then
                    val = uiT[tostring(flag)] and true or false
                else
                    local key = flagMap[flag] or flag
                    if typeof(config[key]) == "boolean" then
                        val = config[key]
                    end
                end
                if typeof(val) == "boolean" then
                    setToggle(toggle, val)
                end
            end
        end

        -- 2) Options / inputs / sliders
        local uiO = type(data._UI) == "table" and data._UI.options or nil
        if type(Options) == "table" then
            for flag, opt in pairs(Options) do
                local val = nil
                if uiO and uiO[tostring(flag)] ~= nil then
                    val = uiO[tostring(flag)]
                else
                    local key = flagMap[flag] or flag
                    val = config[key]
                end
                -- avatar ids especiales
                if flag == "VictimUserId" or flag == "SkinTargetUserId" or flag == "AvatarUserId" then
                    local aid = tonumber(config.SkinTargetUserId)
                    pcall(function()
                        local rs = RivalsSpoof or (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state)
                        if rs and tonumber(rs.avatar_userid) then aid = tonumber(rs.avatar_userid) end
                    end)
                    if data._RS and tonumber(data._RS.avatar_userid) then
                        aid = tonumber(data._RS.avatar_userid)
                    end
                    if aid then val = tostring(aid) end
                end
                if flag == "RivalsDispName" or flag == "DisplayName" then
                    pcall(function()
                        local rs = RivalsSpoof or (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state)
                        if rs and rs.display_name_value then val = tostring(rs.display_name_value) end
                    end)
                end
                if flag == "RivalsUserName" then
                    pcall(function()
                        local rs = RivalsSpoof or (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state)
                        if rs and rs.username_value then val = tostring(rs.username_value) end
                    end)
                end
                if val ~= nil then setOption(opt, val) end
            end
        end
    end)
end

function Cfg.ApplyLoadedState()
    task.spawn(function()
        task.wait(0.2)
        getgenv()._ShakoQuiet = true
        local function safe(fn) pcall(fn) end

        safe(function()
            if config.Enabled or config.StrafeEnabled or config.OrbitEnabled then StartMainLoop()
            else SafeDisconnect("Main"); SafeDisconnect("StickRender") end
        end)
        safe(function()
            if config.RagebotEnabled then
                StartRagebot()
            else
                StopRagebot()
                pcall(function() RunService:UnbindFromRenderStep("__as_rage_restore") end)
            end
        end)
        safe(function()
            if config.VoidSpamEnabled then StartVoidSpam() else StopVoidSpam() end
        end)
        safe(function()
            if config.FlyEnabled then StartFly() else StopFly() end
        end)
        safe(function()
            if config.NoclipEnabled then StartNoclip() else StopNoclip() end
        end)
        safe(function()
            if config.AntiAimEnabled then StartAntiAim() else StopAntiAim() end
        end)
        safe(function()
            if config.AimbotEnabled then StartAimbot() else StopAimbot() end
        end)
        safe(function()
            if config.ESPEnabled then StartESP() else StopESP() end
        end)
        safe(function()
            if config.TriggerbotEnabled then StartTriggerbot() else StopTriggerbot() end
        end)
        safe(function()
            if config.PredDodgeEnabled then StartPredDodge() else StopPredDodge() end
        end)
        safe(function()
            if config.NoRecoil then StartNoRecoil() else StopNoRecoil() end
        end)
        safe(function()
            if config.RapidFire then StartRapidFire() else StopRapidFire() end
        end)
        safe(function()
            if config.FastMelee then StartFastMelee() else StopFastMelee() end
        end)
        safe(function()
            if config.SpeedEnabled then ApplySpeedState() end
            if config.JumpPowerEnabled then ApplyJumpState() end
        end)
        safe(function()
            if config.SlideBoostEnabled then StartSlideBoostLoop() else pcall(StopSlideBoostLoop) end
            if config.SleepyCounterEnabled then StartSleepyCounter() else pcall(StopSleepyCounter) end
            if config.KiciaCounterEnabled then StartKiciaCounter() else pcall(StopKiciaCounter) end
            if config.StickBehindTP then StartStickBehindTP() else pcall(StopStickBehindTP) end
            if config.AmbientSoundEnabled then StartAmbientSound() else pcall(StopAmbientSound) end
        end)
        -- Anti-Rager / Defense: DEBE apagarse al cargar legit
        safe(function()
            if type(anyDefenseOn) == "function" and anyDefenseOn() then
                StartDefense()
            else
                pcall(StopDefense)
                -- forzar flags off si no vienen en config
                if config.RagerCombo == nil then config.RagerCombo = false end
                if not config.OrbitEvade and not config.HeightJitter and not config.AntiHeadTP
                    and not config.StrafePattern and not config.FakePeek and not config.RagerCombo then
                    pcall(StopDefense)
                end
            end
        end)
        safe(function()
            config.StretchedRes = false -- removed
        end)
        -- ===== VISUALES (weather / sky / fullbright / FF) =====
        safe(function()
            -- sync skyboxData desde config guardada
            if skyboxData then
                local preset = config.SkyboxPreset
                if type(preset) == "string" and preset ~= "" and preset ~= "Off" and preset ~= "None" then
                    skyboxData.enabled = true
                    skyboxData.selected = preset
                elseif config.SkyboxEnabled then
                    skyboxData.enabled = true
                    if type(config.SkyboxName) == "string" and config.SkyboxName ~= "" then
                        skyboxData.selected = config.SkyboxName
                    end
                end
                if config.SkyRotate ~= nil then skyboxData.auto_rotate = config.SkyRotate and true or false end
            end
            if skyboxData and skyboxData.enabled then
                pcall(function() ApplySkybox(true) end)
                if skyboxData.auto_rotate and getgenv()._ShakoStartSkyRotate then
                    pcall(function() getgenv()._ShakoStartSkyRotate(true) end)
                end
            end
        end)
        safe(function()
            if config.WeatherEnabled or config.Weather then
                pcall(function() if Config then Config.Weather = true end end)
                pcall(function() if Config then Config.WeatherType = config.WeatherType or "Rain" end end)
                if Weather then
                    if Weather.setType and config.WeatherType then pcall(Weather.setType, config.WeatherType) end
                    if Weather.setIntensity and config.WeatherIntensity then pcall(Weather.setIntensity, config.WeatherIntensity) end
                    if Weather.setSoundVolume and config.WeatherSoundVolume then pcall(Weather.setSoundVolume, config.WeatherSoundVolume) end
                end
                if StartWeather then StartWeather()
                elseif Weather and Weather.enableWeather then Weather.enableWeather() end
            else
                if StopWeather then StopWeather()
                elseif Weather and Weather.disableWeather then Weather.disableWeather() end
            end
        end)
        safe(function()
            if config.SkyboxPreset and config.SkyboxPreset ~= "Off" and Weather and Weather.setSkybox then
                pcall(function() Weather.setSkybox(config.SkyboxPreset) end)
            end
        end)
        safe(function()
            if config.Fullbright then
                pcall(function()
                    if StartFullbright then StartFullbright()
                    else
                        Lighting.Brightness = 2
                        Lighting.ClockTime = 14
                        Lighting.FogEnd = 100000
                        Lighting.GlobalShadows = false
                        Lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)
                    end
                end)
            end
        end)
        safe(function()
            if config.ForceFieldVisual and ApplyForceField and LocalPlayer.Character then
                ApplyForceField(LocalPlayer.Character)
            end
        end)
        safe(function()
            if config.SilentAimEnabled and StartSilentAim then StartSilentAim() end
            if config.SilentShowFOV ~= false and StartFOVDraw then StartFOVDraw() end
        end)
        safe(function()
            if config.ThirdPerson or config.ThirdPersonEnabled then
                if StartThirdPerson then StartThirdPerson() end
            end
        end)
        safe(function()
            if config.CrosshairEnabled and StartCrosshair then StartCrosshair() end
        end)
        safe(function()
            if config.UECounterEnabled or config.AntiUnnamed then
                StartUECounter()
                StartAUWatcher()
            end
        end)
        safe(function()
            if RivalsSpoof and RivalsSpoof.spoof_device then RivalsApplyDevice() end
            if RivalsSpoof and RivalsSpoof.spoof_avatar then RivalsSetAvatarSpoof(true) end
            if RivalsSpoof and RivalsSpoof.display_name then RivalsRefreshNames() end
            if RivalsSpoof and RivalsSpoof.badges then RivalsUpdateBadges() end
        end)
        safe(function()
            RivalsApplyLeaderboard()
            RivalsUpdateCharm()
        end)
        -- re-aplicar unlock skins
        safe(function()
            local st = getgenv()._ShakoStates
            if st and st.skinchanger_state then
                local cd = getgenv()._ShakoModules and getgenv()._ShakoModules.PlayerDataController
                cd = cd and cd.CurrentData
                if cd then
                    pcall(function() cd:Replicate("CosmeticInventory") end)
                    pcall(function() cd:Replicate("WeaponInventory") end)
                end
            end
        end)
        getgenv()._ShakoQuiet = false
        -- un solo aviso, no spam de OFF
        pcall(function()
            Library:Notify({ Title = "Config", Description = "Estado aplicado", Time = 2 })
        end)
    end)
end

function Cfg.Load(name)
    name = Cfg.Sanitize(name)
    if not name or name == "(ninguna)" then
        Notify("Config", "Elige una config")
        return false
    end
    local path = Cfg.folder .. "/" .. name .. ".json"
    if not (isfile and isfile(path)) then
        Notify("Config", "No existe: " .. name)
        return false
    end
    local ok, raw = pcall(function() return readfile(path) end)
    if not ok or not raw or raw == "" then
        Notify("Config", "No se pudo leer")
        return false
    end
    local dok, data = pcall(function() return HttpService:JSONDecode(raw) end)
    if not dok or type(data) ~= "table" then
        Notify("Config", "JSON invalido")
        return false
    end

    -- merge config keys (incluye keys nuevas y boolean false)
    for k, v in pairs(data) do
        if type(k) ~= "string" then continue end
        if k:sub(1, 1) == "_" then continue end -- _RS _UI _Unlock
        local tv = typeof(v)
        if tv ~= "boolean" and tv ~= "number" and tv ~= "string" then continue end
        if config[k] ~= nil then
            local tk = typeof(config[k])
            if tk == tv then
                config[k] = v
            elseif tk == "number" then
                config[k] = tonumber(v) or config[k]
            elseif tk == "boolean" then
                config[k] = v and true or false
            elseif tk == "string" then
                config[k] = tostring(v)
            end
        else
            -- key no estaba en defaults pero es serializable
            config[k] = v
        end
    end

    -- spoof state
    pcall(function()
        local rs = data._RS
        if type(rs) ~= "table" then return end
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        local target = RivalsSpoof or st
        if not target then return end
        if rs.avatar_userid then target.avatar_userid = tonumber(rs.avatar_userid) or target.avatar_userid end
        if rs.spoof_avatar ~= nil then target.spoof_avatar = rs.spoof_avatar and true or false end
        if rs.display_name ~= nil then target.display_name = rs.display_name and true or false end
        if rs.display_name_value then target.display_name_value = tostring(rs.display_name_value) end
        if rs.username ~= nil then target.username = rs.username and true or false end
        if rs.username_value then target.username_value = tostring(rs.username_value) end
        if rs.device then target.device = tostring(rs.device) end
        if rs.spoof_device ~= nil then target.spoof_device = rs.spoof_device and true or false end
        if type(rs.badges) == "table" then
            target.badges = target.badges or {}
            for bk, bv in pairs(rs.badges) do target.badges[bk] = bv and true or false end
        end
        if type(rs.charm) == "table" then
            target.charm = target.charm or {}
            for ck, cv in pairs(rs.charm) do target.charm[ck] = cv end
            if st and st.charm then
                for ck, cv in pairs(rs.charm) do st.charm[ck] = cv end
            end
        end
        if type(rs.leaderboard) == "table" then
            if st and st.leaderboard then
                for lk, lv in pairs(rs.leaderboard) do st.leaderboard[lk] = lv end
            end
            if target.lb then
                target.lb.elo = rs.leaderboard.ELO
                target.lb.elo_value = rs.leaderboard.elo_value
                target.lb.streak = rs.leaderboard.Streak
                target.lb.streak_value = rs.leaderboard.streak_value
                target.lb.kills = rs.leaderboard.Kills
                target.lb.kills_value = rs.leaderboard.kills_value
                target.lb.wins = rs.leaderboard.Wins
                target.lb.wins_value = rs.leaderboard.wins_value
                target.lb.level = rs.leaderboard.Level
                target.lb.level_value = rs.leaderboard.level_value
            end
        end
        if data.SkinTargetUserId then
            config.SkinTargetUserId = tonumber(data.SkinTargetUserId) or config.SkinTargetUserId
        end
    end)

    -- restore unlock skins
    pcall(function()
        local u = data._Unlock
        if type(u) ~= "table" then return end
        local st = getgenv()._ShakoStates
        if not st or not st.skinchanger_state then return end
        if type(u.fake_owned) == "table" then
            st.skinchanger_state.fake_owned = st.skinchanger_state.fake_owned or {}
            for k, v in pairs(u.fake_owned) do
                if v then st.skinchanger_state.fake_owned[k] = true end
            end
        end
        if type(u.fake_weapon_owned) == "table" then
            st.skinchanger_state.fake_weapon_owned = st.skinchanger_state.fake_weapon_owned or {}
            for _, name in ipairs(u.fake_weapon_owned) do
                -- keep as list of names if source expects that
                local found = false
                for _, e in pairs(st.skinchanger_state.fake_weapon_owned) do
                    if e == name or (type(e) == "table" and e.Name == name) then found = true break end
                end
                if not found then
                    table.insert(st.skinchanger_state.fake_weapon_owned, name)
                end
            end
        end
        if type(u.equipped) == "table" then
            st.skinchanger_state.equipped = st.skinchanger_state.equipped or {}
            for k, v in pairs(u.equipped) do st.skinchanger_state.equipped[k] = v end
        end
        if type(u.unclaimed) == "table" and st.inventory_state then
            st.inventory_state.fake_owned = u.unclaimed
        end
        -- replicate so inventory UI updates
        local m = getgenv()._ShakoModules
        local cd = m and m.PlayerDataController and m.PlayerDataController.CurrentData
        if cd then
            pcall(function() cd:Replicate("CosmeticInventory") end)
            pcall(function() cd:Replicate("WeaponInventory") end)
            pcall(function() cd:Replicate("UnclaimedRewards") end)
        end
        Notify("Unlock", "Skins restauradas en config")
    end)

    getgenv()._ObsidianSyncing = true
    -- UI primero (sin callbacks) + reintentos
    Cfg.SyncUIFromConfig(data)
    task.defer(function()
        task.wait(0.2)
        Cfg.SyncUIFromConfig(data)
        task.wait(0.5)
        Cfg.SyncUIFromConfig(data)
        getgenv()._ObsidianSyncing = false
        -- avatar id explícito otra vez
        pcall(function()
            local aid = tonumber(config.SkinTargetUserId)
            if data._RS and tonumber(data._RS.avatar_userid) then
                aid = tonumber(data._RS.avatar_userid)
            end
            if aid then
                config.SkinTargetUserId = aid
                local rs = RivalsSpoof or (getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state)
                if rs then rs.avatar_userid = aid end
                local Options = (Library and Library.Options) or rawget(getgenv(), "Options")
                if type(Options) == "table" then
                    for _, fname in ipairs({"VictimUserId", "SkinTargetUserId", "AvatarUserId"}) do
                        local opt = Options[fname]
                        if opt then
                            pcall(function()
                                if opt.SetValue then opt:SetValue(tostring(aid)) end
                                if opt.SetText then opt:SetText(tostring(aid)) end
                                opt.Value = tostring(aid)
                            end)
                        end
                    end
                end
            end
        end)
    end)
    Cfg.ApplyLoadedState()
    -- segunda pasada visual (weather/sky a veces no pegan a la 1ª)
    task.defer(function()
        task.wait(1.0)
        pcall(function()
            if config.WeatherEnabled or config.Weather then
                pcall(function() if Config then Config.Weather = true end end)
                if Weather and Weather.setType and config.WeatherType then
                    pcall(Weather.setType, config.WeatherType)
                end
                if StartWeather then StartWeather()
                elseif Weather and Weather.enableWeather then Weather.enableWeather() end
            end
            if config.SkyboxPreset and config.SkyboxPreset ~= "Off" and Weather and Weather.setSkybox then
                pcall(Weather.setSkybox, config.SkyboxPreset)
            end
            if skyboxData then
                if config.SkyboxEnabled or (config.SkyboxPreset and config.SkyboxPreset ~= "Off") then
                    skyboxData.enabled = true
                    if config.SkyboxName and config.SkyboxName ~= "" then
                        skyboxData.selected = config.SkyboxName
                    elseif config.SkyboxPreset and config.SkyboxPreset ~= "Off" then
                        -- intentar mapear preset weather a aurora si existe
                        if SkyboxPresets and SkyboxPresets[config.SkyboxPreset] then
                            skyboxData.selected = config.SkyboxPreset
                        end
                    end
                    if ApplySkybox then ApplySkybox(true) end
                end
            end
            if config.Fullbright and Lighting then
                Lighting.Brightness = 2
                Lighting.FogEnd = 1e5
                Lighting.GlobalShadows = false
            end
            if config.ForceFieldVisual and ApplyForceField and LocalPlayer.Character then
                ApplyForceField(LocalPlayer.Character)
            end
        end)
        task.wait(2.0)
        pcall(function()
            if (config.WeatherEnabled or config.Weather) and Weather and Weather.enableWeather then
                local f = workspace:FindFirstChild("_wx")
                if not f then Weather.enableWeather() end
            end
            if skyboxData and skyboxData.enabled and ApplySkybox then ApplySkybox(true) end
        end)
    end)
    Notify("Config", "Cargada: " .. name)
    return true
end

function Cfg.Delete(name)
    name = Cfg.Sanitize(name)
    if not name or name == "(ninguna)" then return false end
    local path = Cfg.folder .. "/" .. name .. ".json"
    if isfile and isfile(path) then
        pcall(function() delfile(path) end)
        Notify("Config", "Borrada: " .. name)
        return true
    end
    return false
end

function Cfg.SetAutoLoad(name)
    name = Cfg.Sanitize(name)
    if not name or name == "(ninguna)" then
        Notify("Config", "Elige o escribe un nombre")
        return false
    end
    if not Cfg.EnsureFolder() then
        Notify("Config", "No hay carpeta configs")
        return false
    end
    local path = Cfg.folder .. "/" .. name .. ".json"
    if not (isfile and isfile(path)) then
        if not Cfg.Save(name, true) then
            Notify("Config", "No se pudo crear " .. name)
            return false
        end
    end
    local ok = pcall(function() writefile(Cfg.autoload, name) end)
    if ok then
        local check = Cfg.GetAutoLoad()
        Notify("Config", "Auto Load ON → " .. tostring(check or name))
        return true
    end
    Notify("Config", "Auto Load write FAIL")
    return false
end

function Cfg.RemoveAutoLoad()
    local ok = pcall(function()
        if isfile and isfile(Cfg.autoload) then delfile(Cfg.autoload) end
    end)
    Notify("Config", ok and "Auto Load OFF" or "No habia Auto Load")
    return ok
end

function Cfg.GetAutoLoad()
    local ok, raw = pcall(function()
        if not (isfile and isfile(Cfg.autoload)) then return nil end
        return readfile(Cfg.autoload)
    end)
    if not ok or not raw then return nil end
    raw = tostring(raw):gsub("%s+", "")
    if raw == "" then return nil end
    return Cfg.Sanitize(raw) or raw
end

function Cfg.TryAutoLoad()
    task.spawn(function()
        for _ = 1, 30 do
            if isfile and isfolder then break end
            task.wait(0.1)
        end
        -- esperar UI (Library.Toggles con flags reales)
        for _ = 1, 40 do
            local T = Library and Library.Toggles
            if type(T) == "table" then
                local n = 0
                for _ in pairs(T) do n = n + 1 if n > 5 then break end end
                if n > 5 then break end
            end
            task.wait(0.15)
        end
        task.wait(0.5)
        if not Cfg.EnsureFolder() then return end
        local name = Cfg.GetAutoLoad()
        if not name then return end
        Notify("Config", "Auto Load: " .. name)
        getgenv()._ObsidianSyncing = true
        local ok = false
        for try = 1, 5 do
            if Cfg.Load(name) then ok = true break end
            task.wait(0.6)
        end
        getgenv()._ObsidianSyncing = false
        Notify("Config", ok and ("Auto Load OK: " .. name) or ("Auto Load FAIL: " .. name))
    end)
end

-- ==================== UI COMPLETA ====================


function BuildUI()
local Tabs = {}

local function bindKey(tog, id, defaultKey)
    pcall(function()
        if not tog or type(tog.AddKeyPicker) ~= "function" then return end
        tog:AddKeyPicker(id, {
            Default = (defaultKey and defaultKey ~= "" and defaultKey) or "...",
            Text = "Keybind",
            Mode = "Toggle",
            Modes = {"Toggle", "Hold", "Always"},
            SyncToggleState = true,
        })
    end)
end

local function addColor(group, id, title, getRGB, setRGB, onChange)
    pcall(function()
        if not group or not group.AddLabel then return end
        local lbl = group:AddLabel(title or id)
        if not lbl or not lbl.AddColorPicker then return end
        local r,g,b = getRGB()
        lbl:AddColorPicker(id, {
            Default = Color3.fromRGB(tonumber(r) or 255, tonumber(g) or 255, tonumber(b) or 255),
            Title = title or id,
            Callback = function(c)
                setRGB(math.floor(c.R*255+0.5), math.floor(c.G*255+0.5), math.floor(c.B*255+0.5))
                if onChange then pcall(onChange, c) end
            end
        })
    end)
end
local function addColor3(group, id, title, getC3, setC3, onChange)
    pcall(function()
        if not group or not group.AddLabel then return end
        local lbl = group:AddLabel(title or id)
        if not lbl or not lbl.AddColorPicker then return end
        lbl:AddColorPicker(id, {
            Default = getC3() or Color3.fromRGB(255,255,255),
            Title = title or id,
            Callback = function(c)
                setC3(c)
                if onChange then pcall(onChange, c) end
            end
        })
    end)
end


local function addTab(key, name, icon)
    local ok, tab = pcall(function() return Window:AddTab(name, icon) end)
    if not ok or not tab then
        ok, tab = pcall(function() return Window:AddTab(name) end)
    end
    if ok and tab then Tabs[key] = tab else warn("[shako] tab fail", name, tab) end
end
-- Tabs compactos (sin HVH aparte — UE/Defense van en Counters)
addTab("Main", "main", "sword")
addTab("World", "world", "globe")
addTab("ESP", "esp", "eye")
addTab("Visual", "visuals", "sparkles")
addTab("Character", "character", "user")
addTab("Spoofer", "spoofers", "user")
addTab("Misc", "misc", "box")
addTab("Settings", "settings", "settings")
if not Tabs.Main then warn("[shako] BuildUI abort: no Main tab") return end
Tabs.Rage = Tabs.Main
Tabs.Void = Tabs.Misc
Tabs.Movement = Tabs.Character
Tabs.Obsidian = Tabs.Misc
Tabs.Configs = Tabs.Settings
if not Tabs.ESP then Tabs.ESP = Tabs.Visual end
if not Tabs.Spoofer then Tabs.Spoofer = Tabs.Misc end
Tabs.Unlock = Tabs.Spoofer


local CombatBox = Tabs.Main:AddLeftGroupbox("Stick / Anti-Sleepy")
local togStick = CombatBox:AddToggle("EnableHVH", {
    Text = "Activar shako", Default = false,
    Callback = function(v)
        if IsAntiUnnamedActive() or config.VoidSpamEnabled then Notify("Warn", "Apaga AU/Void") return end
        config.Enabled = v
        if v then StartMainLoop() else SafeDisconnect("Main") SafeDisconnect("StickRender") CurrentTarget = nil end
    end
})
bindKey(togStick, "StickKey", "...")
CombatBox:AddToggle("TargetLock", {
    Text = "Lock Target", Default = false,
    Callback = function(v) config.TargetLock = v if not v then LockedTarget = nil end end
})
CombatBox:AddDropdown("PriorityMode", {
    Text = "Prioridad", Values = {"Closest", "LowestHP", "Crosshair"}, Default = 1,
    Callback = function(v) config.PriorityMode = v end
})
CombatBox:AddToggle("StickMultiPos", {
    Text = "Multi Pos (cambia direccion)", Default = true,
    Callback = function(v) config.StickMultiPos = v end
})
-- StickDesync siempre ON (dentro del stick, sin opcion separada)
config.StickDesync = true
CombatBox:AddSlider("StickPosInterval", {
    Text = "Cambio pos (s)", Default = 0.18, Min = 0.05, Max = 0.8, Rounding = 2,
    Callback = function(v) config.StickPosInterval = v end
})
CombatBox:AddSlider("StickAway", {
    Text = "Distancia lados", Default = 6, Min = 1, Max = 50, Rounding = 1,
    Callback = function(v) config.StickAway = v end
})
CombatBox:AddToggle("StickFarMode", {
    Text = "Stick LEJOS x2.5 (anti UE)", Default = false,
    Callback = function(v) config.StickFarMode = v end
})
CombatBox:AddToggle("StickFireHead", {
    Text = "Disparar a HEAD (UseItem)", Default = true,
    Callback = function(v) config.StickFireHead = v end
})
CombatBox:AddSlider("StickHeight", {
    Text = "Altura Y (base)", Default = -2.8, Min = -8, Max = 4, Rounding = 1,
    Callback = function(v) config.StickHeight = v end
})
CombatBox:AddSlider("StickOffsetX", {
    Text = "Offset X", Default = 0, Min = -8, Max = 8, Rounding = 1,
    Callback = function(v) config.StickOffsetX = v end
})
CombatBox:AddSlider("StickOffsetZ", {
    Text = "Offset Z", Default = 0, Min = -8, Max = 8, Rounding = 1,
    Callback = function(v) config.StickOffsetZ = v end
})
CombatBox:AddSlider("MaxRange", {
    Text = "Rango (studs)", Default = 999999, Min = 100, Max = 999999, Rounding = 0,
    Callback = function(v) config.MaxRange = v end
})
CombatBox:AddToggle("InfiniteRange", {
    Text = "Rango INFINITO (anti UE)", Default = true,
    Callback = function(v) config.InfiniteRange = v end
})
CombatBox:AddToggle("TeamCheckMain", {
    Text = "Team Check (global)", Default = true,
    Callback = function(v)
        config.TeamCheck = v
        config.RagebotTeamCheck = v
        config.SilentTeamCheck = v
        config.AimbotTeamCheck = v
        config.ESPTeamCheck = v
        Notify("TeamCheck", v and "ON — solo enemigos" or "OFF — todos")
    end
})
CombatBox:AddToggle("UseNameTarget", {
    Text = "Usar Nombre", Default = false,
    Callback = function(v) config.UseNameTarget = v end
})
CombatBox:AddInput("TargetNameInput", {
    Text = "Nombre target", Default = "", Callback = function(v) config.TargetName = v end
})

local SoftBox = Tabs.Main:AddRightGroupbox("Soft TP / Stick extras")
SoftBox:AddToggle("SoftStick", {
    Text = "Soft Stick", Default = true,
    Callback = function(v) config.SoftStick = v end
})
SoftBox:AddSlider("SoftStickSpeed", {
    Text = "Soft Speed", Default = 180, Min = 40, Max = 400, Rounding = 0,
    Callback = function(v) config.SoftStickSpeed = v end
})
SoftBox:AddSlider("SoftStickMaxStep", {
    Text = "Max studs/frame", Default = 5, Min = 1, Max = 20, Rounding = 1,
    Callback = function(v) config.SoftStickMaxStep = v end
})
SoftBox:AddToggle("CameraIndependent", {
    Text = "Camara libre (NO forzar mira)", Default = true,
    Callback = function(v) config.CameraIndependent = v if v then ResetCameraNormal() end end
})
SoftBox:AddLabel("Camara: script no la toca")
SoftBox:AddToggle("Strafe", {
    Text = "Strafe", Default = false,
    Callback = function(v)
        config.StrafeEnabled = v == true
        if v then
            config.OrbitEnabled = false -- nunca al mismo tiempo
            StartMainLoop()
        end
    end
})
SoftBox:AddToggle("Orbit", {
    Text = "Orbit", Default = false,
    Callback = function(v)
        config.OrbitEnabled = v == true
        if v then
            config.StrafeEnabled = false -- nunca al mismo tiempo
            StartMainLoop()
        end
    end
})
SoftBox:AddSlider("OrbitRange", {
    Text = "Rango Orbit", Default = 4, Min = 1, Max = 12, Rounding = 1,
    Callback = function(v) config.OrbitRange = v end
})
SoftBox:AddSlider("OrbitSpeed", {
    Text = "Orbit Speed", Default = 16, Min = 1, Max = 40, Rounding = 0,
    Callback = function(v) config.OrbitSpeed = v end
})
SoftBox:AddSlider("HitboxSize", {
    Text = "Hitbox Size", Default = 11, Min = 3, Max = 20, Rounding = 0,
    Callback = function(v) config.HitboxSize = v end
})
SoftBox:AddToggle("DesyncJitter", {
    Text = "Desync Jitter", Default = false,
    Callback = function(v) config.DesyncJitter = v end
})
SoftBox:AddSlider("SpawnDelay", {
    Text = "Spawn Delay", Default = 4.7, Min = 1, Max = 10, Rounding = 1,
    Callback = function(v) config.SpawnDelay = v end
})
SoftBox:AddToggle("FakePosition", {
    Text = "Fake Position", Default = true,
    Callback = function(v) config.FakePosition = v end
})

-- RAGE FULL
local CounterBox = Tabs.Misc:AddLeftGroupbox("counters")
CounterBox:AddToggle("SleepyCounterEnabled", {
    Text = "Sleepy Counter", Default = false,
    Callback = function(v)
        if v then StartSleepyCounter() else StopSleepyCounter() end
    end
})
CounterBox:AddSlider("SleepyCounterSideDist", {
    Text = "Distancia lado mano", Default = 14, Min = 6, Max = 40, Rounding = 0,
    Callback = function(v) config.SleepyCounterSideDist = v end
})
CounterBox:AddSlider("SleepyCounterHeight", {
    Text = "Altura", Default = 1.2, Min = 0, Max = 8, Rounding = 1,
    Callback = function(v) config.SleepyCounterHeight = v end
})
CounterBox:AddSlider("SleepyCounterFireRate", {
    Text = "Fire rate", Default = 0.02, Min = 0, Max = 0.2, Rounding = 3,
    Callback = function(v) config.SleepyCounterFireRate = v end
})
CounterBox:AddToggle("SleepyCounterTeamCheck", {
    Text = "Team Check", Default = true,
    Callback = function(v) config.SleepyCounterTeamCheck = v end
})
CounterBox:AddDivider()
CounterBox:AddToggle("KiciaCounterEnabled", {
    Text = "Kicia Counter (cielo)", Default = false,
    Callback = function(v)
        if v then StartKiciaCounter() else StopKiciaCounter() end
    end
})
CounterBox:AddSlider("KiciaCounterSkyY", {
    Text = "Altura cielo", Default = 450, Min = 50, Max = 2000, Rounding = 0,
    Callback = function(v) config.KiciaCounterSkyY = v end
})
CounterBox:AddSlider("KiciaCounterLineWidth", {
    Text = "Ancho linea (lado a lado)", Default = 80, Min = 20, Max = 300, Rounding = 0,
    Callback = function(v) config.KiciaCounterLineWidth = v end
})
CounterBox:AddSlider("KiciaCounterMoveSpeed", {
    Text = "Velocidad movimiento", Default = 400, Min = 50, Max = 2500, Rounding = 0,
    Callback = function(v) config.KiciaCounterMoveSpeed = v end
})
CounterBox:AddToggle("KiciaCounterShoot", {
    Text = "Disparar a head desde cielo", Default = true,
    Callback = function(v) config.KiciaCounterShoot = v end
})
CounterBox:AddSlider("KiciaCounterFireRate", {
    Text = "Fire rate (cielo)", Default = 0.015, Min = 0, Max = 0.2, Rounding = 3,
    Callback = function(v) config.KiciaCounterFireRate = v end
})
CounterBox:AddToggle("KiciaCounterTeamCheck", {
    Text = "Team Check", Default = true,
    Callback = function(v) config.KiciaCounterTeamCheck = v end
})
CounterBox:AddLabel("Kicia = TP cielo + linea lateral")
CounterBox:AddLabel("Al OFF vuelve a tu posicion")
CounterBox:AddLabel("Sleepy = lado mano lejitos + head")

local RageBox = Tabs.Main:AddLeftGroupbox("ragebot")
local togRage = RageBox:AddToggle("RagebotToggle", {
    Text = "Ragebot", Default = false,
    Callback = function(v)
        if v and IsAntiUnnamedActive() then Notify("Warn", "Apaga AU") return end
        config.RagebotEnabled = v
        if v then config.RageDesync = true end
        config.RageUseItem = true -- siempre con rage
        if v then StartRagebot() else StopRagebot() end
    end
})
bindKey(togRage, "RagebotKey", "...")
RageBox:AddToggle("RagebotForceFFA", {
    Text = "Force FFA (ignora team)", Default = false,
    Callback = function(v)
        config.RagebotForceFFA = v
        Notify("Rage", v and "FFA ON — ataca a todos" or "FFA auto")
    end
})
RageBox:AddLabel("Free For All: activa Force FFA si no pega")
RageBox:AddToggle("RageIndicatorEnabled", {
    Text = "Rage Indicator (Grief)", Default = true,
    Callback = function(v) config.RageIndicatorEnabled = v end
})
RageBox:AddToggle("RageStatusEnabled", {
    Text = "Rage Status Text", Default = true,
    Callback = function(v) config.RageStatusEnabled = v end
})
RageBox:AddToggle("RageIndicatorSnapline", {
    Text = "Rage Snapline", Default = true,
    Callback = function(v) config.RageIndicatorSnapline = v end
})
RageBox:AddDropdown("RageIndicatorColor", {
    Text = "Indicator Color",
    Values = {"Red", "Purple", "Cyan", "White", "Green"},
    Default = 1,
    Callback = function(v) config.RageIndicatorColor = v end
})
RageBox:AddDropdown("RageStatusMode", {
    Text = "Status Position",
    Values = {"static", "crosshair"},
    Default = 1,
    Callback = function(v) config.RageStatusMode = v end
})
RageBox:AddSlider("RageStatusOffsetY", {
    Text = "Status Offset Y", Default = 48, Min = 0, Max = 200, Rounding = 0,
    Callback = function(v) config.RageStatusOffsetY = v end
})
RageBox:AddDropdown("RageWeaponSlot", {
    Text = "Weapon Slot",
    Values = {"Melee", "Primary", "Secondary"},
    Default = 1,
    Callback = function(v) config.RageWeaponSlot = v end
})
RageBox:AddSlider("RageFireRate", {
    Text = "Fire Rate (0 = max)", Default = 0, Min = 0, Max = 0.05, Rounding = 4,
    Callback = function(v) config.RageFireRate = v end
})
-- Desync integrado en ragebot (siempre ON)
config.RageDesync = true
RageBox:AddToggle("RageSkipDeflect", {
    Text = "Skip Katana Deflect", Default = true,
    Callback = function(v) config.RageSkipDeflect = v end
})
RageBox:AddSlider("RageDesyncKnifeY", {
    Text = "Desync Y (Knife VM)", Default = 6, Min = 1, Max = 12, Rounding = 1,
    Callback = function(v) config.RageDesyncKnifeY = v end
})
RageBox:AddSlider("RageDesyncNormalY", {
    Text = "Desync Y normal", Default = 1, Min = 0, Max = 8, Rounding = 1,
    Callback = function(v) config.RageDesyncNormalY = v end
})
RageBox:AddSlider("RageDesyncNormalZ", {
    Text = "Desync Z normal", Default = 2, Min = 0, Max = 8, Rounding = 1,
    Callback = function(v) config.RageDesyncNormalZ = v end
})
RageBox:AddToggle("RageLookSyncOnShoot", {
    Text = "Mira→Head al disparar (OFF rec.)", Default = false,
    Callback = function(v) config.RageLookSyncOnShoot = v end
})
RageBox:AddButton({
    Text = "Reload Rivals Modules",
    Func = function()
        Notify("Modules", loadRivalsModules() and "OK" or "FAIL — entra a partida")
    end
})
RageBox:AddToggle("VoidRage", {
    Text = "Void+Rage evade", Default = false,
    Callback = function(v) config.VoidRage = v end
})
RageBox:AddDropdown("VoidRageMethods", {
    Text = "Evade tipo",
    Values = {"Side", "Up", "Down", "Back", "Front", "Diagonal", "Orbit", "ZigZag", "HighCircle", "TeleportSpam", "Random", "Circle"},
    Default = 1,
    Callback = function(v) config.VoidRageMethods = v end
})
RageBox:AddSlider("VoidRageAwayDist", {
    Text = "Away dist (hide)", Default = 100000000000, Min = 10, Max = 100000000000, Rounding = 0,
    Callback = function(v) config.VoidRageAwayDist = v end
})
RageBox:AddSlider("VoidRageInterval", {
    Text = "Intervalo", Default = 0.35, Min = 0.1, Max = 1.2, Rounding = 2,
    Callback = function(v) config.VoidRageInterval = v end
})
RageBox:AddToggle("RageVoidFirst", {
    Text = "Void → Attack → Void", Default = true,
    Callback = function(v) config.RageVoidFirst = v end
})
RageBox:AddSlider("RageVoidHideTime", {
    Text = "Hide (tiempo en vacio)", Default = 0.35, Min = 0.28, Max = 3, Rounding = 2,
    Callback = function(v) config.RageVoidHideTime = math.max(v, 0.28) end
})
RageBox:AddSlider("RageVoidAttackTime", {
    Text = "Attack (tiempo pegando)", Default = 0.45, Min = 0.15, Max = 3, Rounding = 2,
    Callback = function(v) config.RageVoidAttackTime = math.max(v, 0.15) end
})
RageBox:AddToggle("RageVoidOnKill", {
    Text = "Void on Kill", Default = true,
    Callback = function(v) config.RageVoidOnKill = v end
})
RageBox:AddToggle("RagePrioritizeRagers", {
    Text = "Prioritize Ragers / Weak", Default = true,
    Callback = function(v) config.RagePrioritizeRagers = v end
})
RageBox:AddLabel("Hide/Attack = ciclo void desync (otros lo ven)")

RageBox:AddSlider("RagebotHeight", {
    Text = "Altura (arriba head)", Default = 2.2, Min = -4, Max = 8, Rounding = 1,
    Callback = function(v) config.RagebotHeight = v end
})
RageBox:AddSlider("RageAttackSpeed", {
    Text = "Attack (vel TP/fire)", Default = 10, Min = 1, Max = 20, Rounding = 1,
    Callback = function(v) config.RageAttackSpeed = v end
})
RageBox:AddLabel("Attack alto = se tpea/pega mas rapido (estilo UE)")
RageBox:AddToggle("RagebotInfiniteRange", {
    Text = "Rango Infinito", Default = true,
    Callback = function(v) config.RagebotInfiniteRange = v end
})
RageBox:AddSlider("RagebotRange", {
    Text = "Rango Rage (si no infinito)", Default = 9999, Min = 50, Max = 99999, Rounding = 0,
    Callback = function(v) config.RagebotRange = v end
})
RageBox:AddSlider("RagebotOffsetX", {
    Text = "Rage Offset X", Default = 0, Min = -10, Max = 10, Rounding = 1,
    Callback = function(v) config.RagebotOffsetX = v end
})
RageBox:AddSlider("RagebotOffsetZ", {
    Text = "Rage Offset Z", Default = 0, Min = -10, Max = 10, Rounding = 1,
    Callback = function(v) config.RagebotOffsetZ = v end
})
RageBox:AddToggle("RagebotTeamCheck", {
    Text = "Team Check (Rage)", Default = true,
    Callback = function(v) config.RagebotTeamCheck = v end
})
RageBox:AddToggle("StickBehindTP", {
    Text = "TP Atras (sin delay)", Default = false,
    Callback = function(v)
        if v then StartStickBehindTP() else StopStickBehindTP() end
    end
})
RageBox:AddSlider("StickBehindDist", {
    Text = "Distancia atras (offset)", Default = 10, Min = 3, Max = 100, Rounding = 0,
    Callback = function(v) config.StickBehindDist = v end
})
RageBox:AddLabel("Alcance TP = infinito (mapa entero)")
RageBox:AddSlider("StickBehindHeight", {
    Text = "Altura atras", Default = 1.5, Min = 0, Max = 12, Rounding = 1,
    Callback = function(v) config.StickBehindHeight = v end
})
RageBox:AddToggle("RagebotPredict", {
    Text = "Predict", Default = true,
    Callback = function(v) config.RagebotPredict = v end
})
RageBox:AddToggle("RageHideLocal", {
    Text = "Hide local body", Default = false,
    Callback = function(v) config.RageHideLocal = v end
})
RageBox:AddSlider("RageShotsPerTick", {
    Text = "Shots / frame", Default = 3, Min = 1, Max = 6, Rounding = 0,
    Callback = function(v) config.RageShotsPerTick = v end
})
RageBox:AddToggle("RageMultipoint", {
    Text = "Multipoint (Head+Torso)", Default = true,
    Callback = function(v) config.RageMultipoint = v end
})
RageBox:AddToggle("RageResolver", {
    Text = "Resolver (anti desync UE)", Default = true,
    Callback = function(v) config.RageResolver = v end
})
RageBox:AddSlider("RageSwitchDelay", {
    Text = "Switch delay (s)", Default = 0.05, Min = 0, Max = 0.5, Rounding = 2,
    Callback = function(v) config.RageSwitchDelay = v end
})
RageBox:AddSlider("RagePredictAmt", {
    Text = "Predict amount", Default = 0.12, Min = 0, Max = 0.5, Rounding = 2,
    Callback = function(v) config.RagebotPredictAmount = v end
})
RageBox:AddButton({
    Text = "PRESET Anti-UE (avanzado)",
    Func = function()
        config.RagebotEnabled = true
        config.RageDesync = true
        config.RageUseItem = true
        config.RagebotInfiniteRange = true
        config.RagebotTeamCheck = true
        config.RagebotPredict = true
        config.RagebotPredictAmount = 0.15
        config.RageFireRate = 0
        config.RageShotsPerTick = 4
        config.RageMultipoint = true
        config.RageResolver = true
        config.RageSkipDeflect = true
        config.RagebotHeight = 2.5
        config.RageDesyncKnifeY = 6
        config.RageDesyncNormalY = 1.2
        config.RageDesyncNormalZ = 2
        config.RageWeaponSlot = "Melee"
        config.VoidRage = true
        config.VoidRageMethods = "Side"
        config.VoidRageAwayDist = 80
        config.VoidRageInterval = 0.25
        config.RageLookSyncOnShoot = false
        config.AntiUEClose = true
        config.DesyncSpam = true
        config.InfiniteRange = true
        pcall(StartRagebot)
        pcall(StartDefense)
        Notify("Rage", "PRESET Anti-UE ON · desync + multipoint + void side")
    end
})
RageBox:AddButton({
    Text = "PRESET Legit-Rage (solo desync)",
    Func = function()
        config.RagebotEnabled = true
        config.RageDesync = true
        config.RageUseItem = true
        config.RagebotInfiniteRange = true
        config.RageShotsPerTick = 2
        config.RageMultipoint = true
        config.VoidRage = false
        config.RageFireRate = 0
        config.RageLookSyncOnShoot = false
        pcall(StartRagebot)
        Notify("Rage", "PRESET Desync only")
    end
})


-- SILENT AIM UI
local SilentBox = Tabs.Main:AddRightGroupbox("silent aim")
local togSilent = SilentBox:AddToggle("SilentAimToggle", {
    Text = "Silent Aim", Default = false,
    Callback = function(v)
        config.SilentAimEnabled = v
        if v then StartSilentAim() else StopSilentAim() end
    end
})
bindKey(togSilent, "SilentAimKey", "...")

SilentBox:AddDropdown("SilentHitPart", {
    Text = "Hit Part",
    Values = {"Head", "HumanoidRootPart", "UpperTorso", "Torso"},
    Default = 1,
    Callback = function(v) config.SilentHitPart = v end
})
SilentBox:AddSlider("SilentFOV", {
    Text = "FOV Radius", Default = 300, Min = 50, Max = 5000, Rounding = 0,
    Callback = function(v) config.SilentFOV = v end
})
SilentBox:AddToggle("SilentPredict", {
    Text = "Predict (anti-atras)", Default = false,
    Callback = function(v) config.SilentPredict = v end
})
SilentBox:AddSlider("SilentPredictAmount", {
    Text = "Predict Amount", Default = 0.06, Min = 0, Max = 0.3, Rounding = 2,
    Callback = function(v) config.SilentPredictAmount = v end
})
SilentBox:AddSlider("SilentHeadOffsetY", {
    Text = "Head Offset Y (arriba)", Default = 0.35, Min = 0, Max = 1.5, Rounding = 2,
    Callback = function(v) config.SilentHeadOffsetY = v end
})

SilentBox:AddToggle("SilentShowFOV", {
    Text = "Show FOV", Default = true,
    Callback = function(v) config.SilentShowFOV = v end
})
SilentBox:AddToggle("SilentFOVFill", {
    Text = "FOV Fill", Default = true,
    Callback = function(v) config.SilentFOVFill = v end
})
SilentBox:AddSlider("SilentFOVFillTransparency", {
    Text = "Fill Transparency", Default = 0.92, Min = 0.5, Max = 0.98, Rounding = 2,
    Callback = function(v) config.SilentFOVFillTransparency = v end
})
SilentBox:AddToggle("SilentFOVAnimated", {
    Text = "Fill Animated", Default = true,
    Callback = function(v) config.SilentFOVAnimated = v end
})
SilentBox:AddToggle("SilentFOVSpin", {
    Text = "FOV Spin (Grief)", Default = true,
    Callback = function(v) config.SilentFOVSpin = v end
})
SilentBox:AddSlider("SilentFOVSpinSpeed", {
    Text = "FOV Spin speed", Default = 1.2, Min = 0.2, Max = 4, Rounding = 1,
    Callback = function(v) config.SilentFOVSpinSpeed = v end
})
SilentBox:AddSlider("SilentFOVThickness", {
    Text = "FOV Outline", Default = 1.5, Min = 1, Max = 4, Rounding = 1,
    Callback = function(v) config.SilentFOVThickness = v end
})
SilentBox:AddDropdown("SilentFOVColor", {
    Text = "FOV Color",
    Values = {"White", "Red", "Green", "Cyan", "Magenta", "Yellow", "Orange"},
    Default = 1,
    Callback = function(v) config.SilentFOVColor = v end
})
SilentBox:AddToggle("SilentTeamCheck", {
    Text = "Team Check", Default = true,
    Callback = function(v) config.SilentTeamCheck = v end
})
SilentBox:AddToggle("SilentForceAiming", {
    Text = "Force Fully Aiming", Default = true,
    Callback = function(v) config.SilentForceAiming = v end
})

local AimBox = Tabs.Main:AddRightGroupbox("aimbot")
local togAim = AimBox:AddToggle("AimbotToggle", {
    Text = "Aimbot (Rivals)", Default = false,
    Callback = function(v) config.AimbotEnabled = v if v then StartAimbot() else StopAimbot() end end
})
bindKey(togAim, "AimbotKey", "...")

AimBox:AddToggle("AimbotKeyHold", {
    Text = "Solo con click derecho (RMB)", Default = true,
    Callback = function(v) config.AimbotKeyHold = v end
})
AimBox:AddToggle("AimbotWallCheck", {
    Text = "Wall Check", Default = true,
    Callback = function(v) config.AimbotWallCheck = v end
})
AimBox:AddToggle("AimbotVisibleCheck", {
    Text = "Visible Check", Default = true,
    Callback = function(v) config.AimbotVisibleCheck = v end
})
AimBox:AddSlider("AimbotFOV", {
    Text = "FOV", Default = 180, Min = 40, Max = 800, Rounding = 0,
    Callback = function(v) config.AimbotFOV = v end
})
AimBox:AddToggle("AimbotShowFOV", {
    Text = "Show FOV", Default = true,
    Callback = function(v) config.AimbotShowFOV = v end
})
AimBox:AddToggle("AimbotFOVFill", {
    Text = "FOV Fill", Default = true,
    Callback = function(v) config.AimbotFOVFill = v end
})
AimBox:AddSlider("AimbotFOVFillTransparency", {
    Text = "Fill Transparency", Default = 0.92, Min = 0.5, Max = 0.98, Rounding = 2,
    Callback = function(v) config.AimbotFOVFillTransparency = v end
})
AimBox:AddToggle("AimbotFOVAnimated", {
    Text = "Fill Animated", Default = true,
    Callback = function(v) config.AimbotFOVAnimated = v end
})
AimBox:AddDropdown("AimbotFOVColor", {
    Text = "FOV Color",
    Values = {"White", "Red", "Green", "Cyan", "Magenta", "Yellow", "Orange"},
    Default = 1,
    Callback = function(v) config.AimbotFOVColor = v end
})
AimBox:AddSlider("AimbotSmooth", {
    Text = "Smooth", Default = 0.15, Min = 0.02, Max = 1, Rounding = 2,
    Callback = function(v) config.AimbotSmooth = v end
})
AimBox:AddDropdown("AimbotPart", {
    Text = "Part", Values = {"Head", "HumanoidRootPart", "UpperTorso"}, Default = 1,
    Callback = function(v) config.AimbotPart = v end
})
AimBox:AddToggle("AimbotTeamCheckUI", {
    Text = "Team Check", Default = true,
    Callback = function(v) config.AimbotTeamCheck = v end
})
AimBox:AddToggle("AutoClickToggle", {
    Text = "Auto Clicker", Default = false,
    Callback = function(v) config.AutoClickEnabled = v if v then StartAutoClick() else StopAutoClick() end end
})
AimBox:AddSlider("AutoClickCPS", {
    Text = "CPS", Default = 16, Min = 1, Max = 30, Rounding = 0,
    Callback = function(v) config.AutoClickCPS = v end
})
AimBox:AddToggle("AutoClickOnlyOnTarget", {
    Text = "Click solo con target", Default = true,
    Callback = function(v) config.AutoClickOnlyOnTarget = v end
})
AimBox:AddToggle("NoRecoilToggle", {
    Text = "No Recoil", Default = true,
    Callback = function(v) config.NoRecoil = v if v then StartNoRecoil() else StopNoRecoil() end end
})
AimBox:AddToggle("NoRecoilOnlyShoot", {
    Text = "Solo al disparar", Default = true,
    Callback = function(v) config.NoRecoilOnlyShoot = v end
})
AimBox:AddLabel("No Recoil = ItemLibrary (sin zoom)")
AimBox:AddSlider("NoRecoilStrength", {
    Text = "Fuerza NR", Default = 1, Min = 0.1, Max = 1, Rounding = 2,
    Callback = function(v) config.NoRecoilStrength = v end
})
AimBox:AddToggle("RapidFireToggle", {
    Text = "Fast Fire (ItemLibrary)", Default = false,
    Callback = function(v) config.RapidFire = v if v then StartRapidFire() else StopRapidFire() end end
})
AimBox:AddToggle("FastMeleeToggle", {
    Text = "Fast Melee (ItemLibrary)", Default = false,
    Callback = function(v) if v then StartFastMelee() else StopFastMelee() end end
})
AimBox:AddSlider("RapidFireDelay", {
    Text = "ShootCooldown", Default = 0.001, Min = 0.0001, Max = 0.05, Rounding = 4,
    Callback = function(v) config.RapidFireDelay = v if config.RapidFire then ApplyItemLibraryGuns(true) end end
})
AimBox:AddToggle("ItemLibraryKeepAlive", {
    Text = "Keep Alive ItemLibrary", Default = true,
    Callback = function(v) config.ItemLibraryKeepAlive = v end
})
AimBox:AddButton({
    Text = "Test ItemLibrary",
    Func = function()
        local items = getItems()
        if not items then Notify("Lib", "FAIL") return end
        local n = 0
        for _ in pairs(items) do n += 1 end
        Notify("Lib", n .. " items")
    end
})
AimBox:AddLabel("No Recoil = ItemLibrary ShootRecoil=0")

-- VOID
local VoidBox = Tabs.Misc:AddLeftGroupbox("void")
local togVoid = VoidBox:AddToggle("VoidSpam", {
    Text = "Enable Void Spam", Default = false,
    Callback = function(v)
        if v and IsAntiUnnamedActive() then Notify("AU", "Apaga AU") return end
        config.VoidSpamEnabled = v
        if v then
            config.Enabled = false
            if not config.VoidRage then config.RagebotEnabled = false end
            SafeDisconnect("Main")
            SafeDisconnect("StickRender")
            if not config.VoidRage then SafeDisconnect("Ragebot") end
            StartVoidSpam()
        else
            StopVoidSpam()
        end
    end
})
bindKey(togVoid, "VoidSpamKey", "...")

VoidBox:AddDropdown("VoidMethod", {
    Text = "Metodo",
    Values = {"Quantum", "Drift", "Chaos", "Loop", "Spiral"},
    Default = 1,
    Callback = function(v) config.VoidMethod = v end
})
VoidBox:AddToggle("VoidEvade", {
    Text = "Void Evade (ultra lejos)", Default = true,
    Callback = function(v) config.VoidEvade = v end
})
VoidBox:AddSlider("VoidEvadeDistUI", {
    Text = "Evade Dist", Default = 100000000000, Min = 1000, Max = 100000000000, Rounding = 0,
    Callback = function(v)
        config.VoidEvadeDist = v
    end
})
VoidBox:AddLabel("Distancia void spam / evade")
VoidBox:AddSlider("VoidSpeedUI", {
    Text = "Speed", Default = 100000000000, Min = 1, Max = 100000000000, Rounding = 0,
    Callback = function(v) config.VoidSpeed = v end
})
VoidBox:AddSlider("VoidAltUI", {
    Text = "Altitude", Default = 100000000000, Min = 1000, Max = 100000000000, Rounding = 0,
    Callback = function(v)
        config.VoidBaseAltitude = v
        config.VoidRadius = math.max(v, config.VoidRadius or v)
    end
})
VoidBox:AddSlider("VoidRadiusUI", {
    Text = "Radius", Default = 100000000000, Min = 1000, Max = 100000000000, Rounding = 0,
    Callback = function(v) config.VoidRadius = v end
})

local DodgeBox = Tabs.Misc:AddRightGroupbox("dodge")
DodgeBox:AddToggle("PredDodge", {
    Text = "Prediction Dodge", Default = false,
    Callback = function(v) config.PredDodgeEnabled = v if v then StartPredDodge() else StopPredDodge() end end
})
DodgeBox:AddSlider("PredRadius", {
    Text = "Danger Radius", Default = 20, Min = 5, Max = 100, Rounding = 0,
    Callback = function(v) config.PredRadius = v end
})
DodgeBox:AddSlider("PredDodgeDist", {
    Text = "Dodge Dist", Default = 30, Min = 5, Max = 150, Rounding = 0,
    Callback = function(v) config.PredDodgeDist = v end
})
DodgeBox:AddToggle("Triggerbot", {
    Text = "Triggerbot", Default = false,
    Callback = function(v) config.TriggerbotEnabled = v if v then StartTriggerbot() else StopTriggerbot() end end
})
DodgeBox:AddSlider("TriggerbotFOV", {
    Text = "Trigger FOV", Default = 40, Min = 5, Max = 120, Rounding = 0,
    Callback = function(v) config.TriggerbotFOV = v end
})
DodgeBox:AddSlider("TriggerbotCPS", {
    Text = "Trigger CPS", Default = 12, Min = 1, Max = 25, Rounding = 0,
    Callback = function(v) config.TriggerbotCPS = v end
})

-- VISUAL FULL
local VisBox = Tabs.World:AddLeftGroupbox("lighting")
VisBox:AddToggle("ForceFieldVisual", {
    Text = "ForceField visual", Default = false,
    Callback = function(v)
        config.ForceFieldVisual = v
        if v then StartForceField() else StopForceField() end
    end
})
VisBox:AddDropdown("ForceFieldStyle", {
    Text = "FF Estilo",
    Values = {"Highlight", "Soft", "Material", "Both"},
    Default = 1,
    Callback = function(v)
        config.ForceFieldStyle = v
        if config.ForceFieldVisual then ApplyForceField() end
    end
})
VisBox:AddSlider("ForceFieldTransparency", {
    Text = "FF Fill (más alto = más transparente)", Default = 0.72, Min = 0.1, Max = 0.95, Rounding = 2,
    Callback = function(v) config.ForceFieldTransparency = v if config.ForceFieldVisual then ApplyForceField() end end
})
VisBox:AddSlider("ForceFieldOutlineTransparency", {
    Text = "FF Outline transparency", Default = 0.55, Min = 0, Max = 0.95, Rounding = 2,
    Callback = function(v) config.ForceFieldOutlineTransparency = v if config.ForceFieldVisual then ApplyForceField() end end
})
VisBox:AddToggle("ForceFieldPulse", {
    Text = "FF Pulse (animado)", Default = false,
    Callback = function(v)
        config.ForceFieldPulse = v
        if config.ForceFieldVisual then StartForceField() end
    end
})
addColor3(VisBox, "ForceFieldColorPick", "ForceField Color",
    function() return config.ForceFieldColor or Color3.fromRGB(180,0,255) end,
    function(c) config.ForceFieldColor = c end,
    function() if config.ForceFieldVisual then ApplyForceField() end end)
VisBox:AddDropdown("ForceFieldColorPreset", {
    Text = "FF Color preset",
    Values = {"Morado", "Cyan", "Rojo", "Verde", "Blanco", "Rosa"},
    Default = 1,
    Callback = function(v)
        local map = {
            Morado = Color3.fromRGB(180, 0, 255),
            Cyan = Color3.fromRGB(0, 220, 255),
            Rojo = Color3.fromRGB(255, 40, 60),
            Verde = Color3.fromRGB(40, 255, 120),
            Blanco = Color3.fromRGB(255, 255, 255),
            Rosa = Color3.fromRGB(255, 80, 180),
        }
        config.ForceFieldColor = map[v] or map.Morado
        if config.ForceFieldVisual then ApplyForceField() end
    end
})
VisBox:AddToggle("ForceFieldRemoveAcc", {
    Text = "FF modo agresivo (no recomendado)", Default = false,
    Callback = function(v) config.ForceFieldRemoveAcc = v end
})
VisBox:AddToggle("ThirdPerson", {
    Text = "Third person (LuaHook)", Default = false,
    Callback = function(v)
        if v then StartThirdPerson() else StopThirdPerson() end
    end
})
VisBox:AddSlider("ThirdPersonDistance", {
    Text = "TP Distance", Default = 12, Min = 4, Max = 30, Rounding = 0,
    Callback = function(v) config.ThirdPersonDistance = v end
})

VisBox:AddSlider("ThirdPersonHeight", {
    Text = "TP Altura", Default = 2.5, Min = 0, Max = 10, Rounding = 1,
    Callback = function(v) config.ThirdPersonHeight = v end
})
VisBox:AddToggle("Fullbright", {
    Text = "Fullbright", Default = false,
    Callback = function(v) config.Fullbright = v ApplyVisuals() end
})
VisBox:AddToggle("LightingOverride", {
    Text = "Enable Lighting Override", Default = false,
    Callback = function(v) config.LightingOverride = v ApplyVisuals() end
})
VisBox:AddSlider("Brightness", {
    Text = "Brightness", Default = 2, Min = 0, Max = 10, Rounding = 1,
    Callback = function(v) config.Brightness = v ApplyVisuals() end
})
VisBox:AddSlider("Exposure", {
    Text = "Exposure", Default = 0, Min = -3, Max = 3, Rounding = 1,
    Callback = function(v) config.Exposure = v ApplyVisuals() end
})
VisBox:AddSlider("ClockTime", {
    Text = "Time (Clock)", Default = 14, Min = 0, Max = 24, Rounding = 0,
    Callback = function(v) config.ClockTime = v ApplyVisuals() end
})
VisBox:AddToggle("NoFog", {
    Text = "No Fog", Default = true,
    Callback = function(v) config.NoFog = v ApplyVisuals() end
})
VisBox:AddToggle("GlobalShadows", {
    Text = "Global Shadows", Default = true,
    Callback = function(v) config.GlobalShadows = v ApplyVisuals() end
})
VisBox:AddToggle("AmbientColor", {
    Text = "Ambient Colors", Default = false,
    Callback = function(v) config.AmbientColor = v ApplyVisuals() end
})
addColor(VisBox, "AmbientColorPick", "Ambient",
    function() return config.AmbientR or 120, config.AmbientG or 120, config.AmbientB or 140 end,
    function(r,g,b) config.AmbientR, config.AmbientG, config.AmbientB = r,g,b end,
    function() if config.AmbientColor or config.LightingOverride then ApplyVisuals() end end)
addColor(VisBox, "OutdoorColorPick", "Outdoor Ambient",
    function() return config.OutdoorR or 140, config.OutdoorG or 140, config.OutdoorB or 160 end,
    function(r,g,b) config.OutdoorR, config.OutdoorG, config.OutdoorB = r,g,b end,
    function() if config.AmbientColor or config.LightingOverride then ApplyVisuals() end end)
VisBox:AddToggle("ColorShiftEnabled", {
    Text = "Color Shift", Default = false,
    Callback = function(v) config.ColorShiftEnabled = v ApplyVisuals() end
})
addColor(VisBox, "ColorShiftPick", "Color Shift",
    function() return config.ColorShiftR or 0, config.ColorShiftG or 0, config.ColorShiftB or 0 end,
    function(r,g,b) config.ColorShiftR, config.ColorShiftG, config.ColorShiftB = r,g,b end,
    function() if config.ColorShiftEnabled then ApplyVisuals() end end)
VisBox:AddToggle("TintEnabled", {
    Text = "Tint Color", Default = false,
    Callback = function(v) config.TintEnabled = v ApplyVisuals() end
})
addColor(VisBox, "TintColorPick", "Tint Color",
    function() return config.TintR or 255, config.TintG or 255, config.TintB or 255 end,
    function(r,g,b) config.TintR, config.TintG, config.TintB = r,g,b end,
    function() if config.TintEnabled then ApplyVisuals() end end)
VisBox:AddSlider("Saturation", {
    Text = "Saturation", Default = 0, Min = -1, Max = 1, Rounding = 2,
    Callback = function(v) config.Saturation = v ApplyVisuals() end
})
VisBox:AddSlider("Contrast", {
    Text = "Contrast", Default = 0, Min = -1, Max = 1, Rounding = 2,
    Callback = function(v) config.Contrast = v ApplyVisuals() end
})
VisBox:AddToggle("BloomEnabled", {
    Text = "Bloom", Default = false,
    Callback = function(v) config.BloomEnabled = v ApplyVisuals() end
})
VisBox:AddSlider("BloomIntensity", {
    Text = "Bloom Intensity", Default = 0.4, Min = 0, Max = 2, Rounding = 2,
    Callback = function(v) config.BloomIntensity = v ApplyVisuals() end
})
VisBox:AddToggle("BlurEnabled", {
    Text = "Blur", Default = false,
    Callback = function(v) config.BlurEnabled = v ApplyVisuals() end
})
VisBox:AddSlider("BlurSize", {
    Text = "Blur Size", Default = 8, Min = 1, Max = 24, Rounding = 0,
    Callback = function(v) config.BlurSize = v ApplyVisuals() end
})
VisBox:AddDivider()
VisBox:AddLabel("—— World / Map ——")
VisBox:AddToggle("WallXray", {
    Text = "Wall X-Ray (transparencia)", Default = false,
    Callback = function(v)
        config.WallXray = v
        if v then
            ApplyVisuals()
        else
            pcall(function() applyWallXray(false) end)
        end
        Notify("Visual", v and "Wall X-Ray ON" or "Wall X-Ray OFF")
    end
})
VisBox:AddSlider("WallTransparency", {
    Text = "Wall Transparency", Default = 0.55, Min = 0.1, Max = 0.95, Rounding = 2,
    Callback = function(v)
        config.WallTransparency = v
        if config.WallXray then pcall(function() applyWallXray(true) end) end
    end
})
VisBox:AddToggle("MapColorEnabled", {
    Text = "Map Color (tinte mapa)", Default = false,
    Callback = function(v)
        config.MapColorEnabled = v
        if v then ApplyVisuals() else pcall(function() applyMapColor(false) end) end
    end
})
addColor(VisBox, "MapColorPick", "Map Tint",
    function() return config.MapColorR or 80, config.MapColorG or 80, config.MapColorB or 100 end,
    function(r,g,b) config.MapColorR, config.MapColorG, config.MapColorB = r,g,b end,
    function() if config.MapColorEnabled then pcall(function() applyMapColor(true) end) end end)
VisBox:AddToggle("RemoveTextures", {
    Text = "Remove Textures", Default = false,
    Callback = function(v)
        config.RemoveTextures = v
        if v then pcall(function() applyRemoveTextures(true) end) end
        Notify("Visual", v and "Textures off" or "Textures (solo al ON)")
    end
})
VisBox:AddButton({
    Text = "Preset HVH",
    Func = function()
        config.Fullbright = true
        config.LightingOverride = true
        config.NoFog = true
        config.Saturation = 0.1
        config.GlobalShadows = false
        ApplyVisuals()
        ResetCameraNormal()
    end
})
VisBox:AddButton({
    Text = "Preset Night Blue",
    Func = function()
        config.LightingOverride = true
        config.Fullbright = false
        config.ClockTime = 22
        config.Brightness = 1.5
        config.AmbientR, config.AmbientG, config.AmbientB = 40, 50, 90
        config.OutdoorR, config.OutdoorG, config.OutdoorB = 30, 40, 80
        config.ColorShiftEnabled = true
        config.ColorShiftR, config.ColorShiftG, config.ColorShiftB = 20, 40, 120
        config.NoFog = true
        ApplyVisuals()
    end
})
VisBox:AddButton({
    Text = "Reset Camara",
    Func = function()
        ResetCameraNormal()
        EnsureCameraFree()
        Notify("Cam", "Custom OK")
    end
})
VisBox:AddButton({ Text = "Reset Visual", Func = ResetVisuals })

local ESPBox = Tabs.ESP:AddLeftGroupbox("ESP")
local togESP = ESPBox:AddToggle("ESPEnabled", {
    Text = "ESP (LuaHook)", Default = false,
    Callback = function(v)
        config.ESPEnabled = v
        config.ESP = v
        if v then StartESP() else StopESP() end
    end
})
bindKey(togESP, "ESPKey", "...")

ESPBox:AddToggle("ESPPerfMode", {
    Text = "ESP High FPS (recomendado)", Default = true,
    Callback = function(v)
        config.ESPPerfMode = v
        if v then
            config.ESPSkeleton = false
            config.ESPRadar = false
            config.ESPChams = false
            config.ESPTracers = false
            config.ESPMaxPlayers = math.min(tonumber(config.ESPMaxPlayers) or 8, 8)
        end
    end
})
ESPBox:AddSlider("ESPUpdateInterval", {
    Text = "ESP refresh (s) · mas = mas FPS", Default = 0.1, Min = 0.05, Max = 0.25, Rounding = 2,
    Callback = function(v) config.ESPUpdateInterval = v end
})
ESPBox:AddSlider("ESPMaxPlayers", {
    Text = "Max jugadores ESP", Default = 8, Min = 4, Max = 20, Rounding = 0,
    Callback = function(v) config.ESPMaxPlayers = v end
})
ESPBox:AddToggle("ESPTeamCheck", {
    Text = "Team Check (solo enemigos)", Default = true,
    Callback = function(v) config.ESPTeamCheck = v end
})
addColor3(ESPBox, "ESPBoxColorPick", "ESP Box Color",
    function() return config.ESPBoxColor or Color3.fromRGB(255,255,255) end,
    function(c) config.ESPBoxColor = c end, function() end)
addColor3(ESPBox, "ESPFillColorPick", "ESP Fill Color",
    function() return config.ESPBoxFillColor or Color3.fromRGB(255,59,78) end,
    function(c) config.ESPBoxFillColor = c end, function() end)
ESPBox:AddDropdown("ESPColor", {
    Text = "Color ESP",
    Values = {"White", "Red", "Green", "Blue", "Cyan", "Yellow", "Magenta", "Orange"},
    Default = 1,
    Callback = function(v) config.ESPColor = v end
})
ESPBox:AddDropdown("ESPHealthBarColor", {
    Text = "Color Healthbar",
    Values = {"Green", "White", "Red", "Blue", "Cyan", "Yellow", "Magenta", "Orange"},
    Default = 1,
    Callback = function(v) config.ESPHealthBarColor = v end
})
ESPBox:AddToggle("ESPUseTeamColor", {
    Text = "Color por team",
    Default = false,
    Callback = function(v) config.ESPUseTeamColor = v end
})
ESPBox:AddToggle("ESPUseHPColor", {
    Text = "Color por vida",
    Default = false,
    Callback = function(v) config.ESPUseHPColor = v end
})
ESPBox:AddToggle("ESPTracers", {
    Text = "Tracers", Default = true,
    Callback = function(v) config.ESPTracers = v end
})
ESPBox:AddDropdown("ESPTracerFrom", {
    Text = "Tracer from",
    Values = {"Bottom", "Center", "Mouse"},
    Default = 1,
    Callback = function(v) config.ESPTracerFrom = v end
})
ESPBox:AddToggle("ESPShowName", {
    Text = "Name", Default = true,
    Callback = function(v) config.ESPShowName = v; config.ESPName = v end
})
ESPBox:AddToggle("ESPShowDist", {
    Text = "Distance", Default = true,
    Callback = function(v) config.ESPShowDist = v; config.ESPDistance = v end
})
ESPBox:AddToggle("ESPShowHP", {
    Text = "HP / Healthbar", Default = true,
    Callback = function(v) config.ESPShowHP = v; config.ESPHealth = v; config.ESPHealthBar = v end
})
ESPBox:AddToggle("ESPShowBox", {
    Text = "Box", Default = true,
    Callback = function(v)
        config.ESPShowBox = v
        config.ESPBox = v
    end
})
ESPBox:AddDropdown("ESPBoxStyle", {
    Text = "Box Style",
    Values = {"Full Box", "Corner Brackets"},
    Default = 1,
    Callback = function(v)
        config.ESPBoxStyle = v
        config.ESPBoxBrackets = (v == "Corner Brackets")
        if ESP and ESP.rebuildAll then pcall(ESP.rebuildAll) end
    end
})
ESPBox:AddToggle("ESPSkeleton", {
    Text = "Skeleton", Default = true,
    Callback = function(v) config.ESPSkeleton = v end
})
ESPBox:AddToggle("ESPGlow", {
    Text = "Glow (Highlight)", Default = true,
    Callback = function(v) config.ESPGlow = v end
})
ESPBox:AddToggle("ESPOutline", {
    Text = "Outline (calidad pro)", Default = true,
    Callback = function(v) config.ESPOutline = v end
})
ESPBox:AddToggle("ESPChams", {
    Text = "Chams (Highlight)", Default = false,
    Callback = function(v) config.ESPChams = v end
})
ESPBox:AddSlider("ESPChamsFill", {
    Text = "Chams Fill Trans", Default = 0.55, Min = 0, Max = 1, Rounding = 2,
    Callback = function(v) config.ESPChamsFill = v end
})
ESPBox:AddToggle("ESPHeadDot", {
    Text = "Head Dot", Default = true,
    Callback = function(v) config.ESPHeadDot = v end
})
ESPBox:AddToggle("ESPShowWeapon", {
    Text = "Weapon Name", Default = true,
    Callback = function(v) config.ESPShowWeapon = v end
})
ESPBox:AddToggle("ESPShowHPText", {
    Text = "HP Text / %", Default = true,
    Callback = function(v) config.ESPShowHPText = v end
})
ESPBox:AddToggle("ESPUseHPColor", {
    Text = "Color por HP", Default = true,
    Callback = function(v) config.ESPUseHPColor = v end
})
ESPBox:AddSlider("ESPBoxThickness", {
    Text = "Box Thickness", Default = 1.8, Min = 0.5, Max = 4, Rounding = 1,
    Callback = function(v) config.ESPBoxThickness = v end
})
ESPBox:AddSlider("ESPSkeletonThickness", {
    Text = "Skeleton Thickness", Default = 1.8, Min = 0.5, Max = 4, Rounding = 1,
    Callback = function(v) config.ESPSkeletonThickness = v end
})
ESPBox:AddSlider("ESPNameSize", {
    Text = "Name Size", Default = 15, Min = 10, Max = 22, Rounding = 0,
    Callback = function(v) config.ESPNameSize = v end
})
ESPBox:AddSlider("ESPBarWidth", {
    Text = "HP Bar Width", Default = 4, Min = 2, Max = 8, Rounding = 0,
    Callback = function(v) config.ESPBarWidth = v end
})
ESPBox:AddButton({
    Text = "PRESET ESP Experto",
    Func = function()
        config.ESPEnabled = true
        config.ESPShowBox = true
        config.ESPBoxStyle = "Corner"
        config.ESPSkeleton = true
        config.ESPOutline = true
        config.ESPHeadDot = true
        config.ESPShowName = true
        config.ESPShowDist = true
        config.ESPShowHP = true
        config.ESPShowHPText = true
        config.ESPShowWeapon = true
        config.ESPUseHPColor = true
        config.ESPInfiniteRange = true
        config.ESPTeamCheck = true
        config.ESPBoxThickness = 1.8
        config.ESPSkeletonThickness = 1.8
        config.ESPBarWidth = 4
        config.ESPNameSize = 15
        config.ESPColor = "White"
        config.ESPHealthBarColor = "Green"
        config.ESPTracers = false
        config.ESPChams = false
        pcall(StartESP)
        Notify("ESP", "PRESET Experto ON")
    end
})
ESPBox:AddToggle("ESPInfiniteRange", {
    Text = "ESP Rango Infinito", Default = true,
    Callback = function(v) config.ESPInfiniteRange = v end
})
ESPBox:AddToggle("ESPFilledBox", {
    Text = "Box Filled", Default = false,
    Callback = function(v) config.ESPFilledBox = v end
})
ESPBox:AddToggle("BulletTracerEnabled", {
    Text = "Bullet Tracers", Default = true,
    Callback = function(v) config.BulletTracerEnabled = v end
})
addColor3(ESPBox, "BulletTracerColorPick", "Bullet Tracer Color",
    function() return config.BulletTracerColorCustom or Color3.fromRGB(0,220,255) end,
    function(c) config.BulletTracerColorCustom = c; config.BulletTracerColor = "Custom" end, function() end)
ESPBox:AddDropdown("BulletTracerColor", {
    Text = "Bullet Tracer Color",
    Values = {"Cyan", "White", "Red", "Green", "Yellow", "Magenta", "Orange", "Blue"},
    Default = 1,
    Callback = function(v) config.BulletTracerColor = v end
})
ESPBox:AddSlider("BulletTracerDuration", {
    Text = "Tracer Duration", Default = 1.2, Min = 0.2, Max = 5, Rounding = 1,
    Callback = function(v) config.BulletTracerDuration = v end
})
ESPBox:AddDropdown("BulletTracerStyle", {
    Text = "Tracer style (Grief)",
    Values = {"Line", "Laser", "Glow"},
    Default = 1,
    Callback = function(v) config.BulletTracerStyle = v end
})
ESPBox:AddSlider("BulletTracerGlow", {
    Text = "Tracer glow", Default = 0.45, Min = 0, Max = 1, Rounding = 2,
    Callback = function(v) config.BulletTracerGlow = v end
})

ESPBox:AddSlider("ESPMaxDist", {
    Text = "Max Dist", Default = 600, Min = 50, Max = 2000, Rounding = 0,
    Callback = function(v) config.ESPMaxDist = v end
})
-- crosshair moved to visuals
ESPBox:AddSlider("CharacterTransparency"
, {
    Text = "Tu transparencia", Default = 0, Min = 0, Max = 1, Rounding = 2,
    Callback = function(v) config.CharacterTransparency = v end
})
ESPBox:AddToggle("RemoveAccessories", {
    Text = "Quitar accesorios", Default = false,
    Callback = function(v) config.RemoveAccessories = v end
})

local SkyBox = Tabs.World:AddLeftGroupbox("Skybox")
local skyNames = {"None"}
pcall(function()
    for n in pairs(Skies or {}) do table.insert(skyNames, n) end
end)
-- AURORA presets
local auroraNames = {}
pcall(function()
    local presets = getgenv()._ShakoSkyPresets
    if type(presets) == "table" then
        for n in pairs(presets) do
            if n ~= "None" then table.insert(auroraNames, "AURORA:" .. n) end
        end
        table.sort(auroraNames)
        for _, n in ipairs(auroraNames) do table.insert(skyNames, n) end
    end
end)
SkyBox:AddDropdown("SkySelect", {
    Text = "Cielo", Values = skyNames, Default = 1,
    Callback = function(v)
        if type(v) == "string" and v:sub(1,7) == "AURORA:" then
            local name = v:sub(8)
            local sd = getgenv()._ShakoSkyData
            if sd then sd.enabled = true; sd.selected = name end
            if getgenv()._ShakoApplySkybox then
                pcall(getgenv()._ShakoApplySkybox)
            end
            Notify("Sky", "AURORA → " .. name)
        else
            pcall(ApplySky, v)
        end
    end
})
SkyBox:AddToggle("AuroraSkyEnable", {
    Text = "AURORA Sky Enabled", Default = false,
    Callback = function(v)
        local sd = getgenv()._ShakoSkyData
        if sd then sd.enabled = v end
        if v and getgenv()._ShakoApplySkybox then pcall(getgenv()._ShakoApplySkybox) end
        Notify("Sky", v and "ON" or "OFF")
    end
})
SkyBox:AddToggle("AuroraSkyRotate", {
    Text = "Auto Rotate", Default = false,
    Callback = function(v)
        if getgenv()._ShakoStartSkyRotate then pcall(getgenv()._ShakoStartSkyRotate, v) end
    end
})
SkyBox:AddSlider("AuroraSkyStars", {
    Text = "Stars", Default = 3000, Min = 0, Max = 5000, Rounding = 0,
    Callback = function(v)
        local sd = getgenv()._ShakoSkyData
        if sd then sd.star_count = v end
        if getgenv()._ShakoApplySkybox then pcall(getgenv()._ShakoApplySkybox) end
    end
})

local WeatherBox = Tabs.World:AddRightGroupbox("Weather")
WeatherBox:AddToggle("WeatherEnabled", {
    Text = "Enable Weather", Default = false,
    Callback = function(v)
        config.WeatherEnabled = v
        if v then StartWeather() else StopWeather() end
    end
})
WeatherBox:AddDropdown("WeatherType", {
    Text = "Type",
    Values = {"Rain", "Snow", "Mist", "Embers", "Fireflies", "Petals", "Autumn", "Ash", "Sandstorm"},
    Default = 1,
    Callback = function(v)
        config.WeatherType = v
        pcall(function() if Weather and Weather.setType then Weather.setType(v) end end)
        if config.WeatherEnabled then StartWeather() end
    end
})
WeatherBox:AddSlider("WeatherIntensity", {
    Text = "Intensity", Default = 1, Min = 0.15, Max = 2, Rounding = 2,
    Callback = function(v)
        config.WeatherIntensity = v
        pcall(function() if Weather and Weather.setIntensity then Weather.setIntensity(v) end end)
    end
})
WeatherBox:AddSlider("WeatherSoundVolume", {
    Text = "Sound volume", Default = 0.6, Min = 0, Max = 1, Rounding = 2,
    Callback = function(v)
        config.WeatherSoundVolume = v
        pcall(function() if Weather and Weather.setSoundVolume then Weather.setSoundVolume(v) end end)
    end
})
WeatherBox:AddToggle("WeatherStorm", {
    Text = "Storm (thunder)", Default = false,
    Callback = function(v)
        config.WeatherStorm = v
        pcall(function() if Weather and Weather.toggleStorm then Weather.toggleStorm(v) end end)
    end
})
WeatherBox:AddSlider("WeatherStormMin", {
    Text = "Storm interval min", Default = 8, Min = 1, Max = 30, Rounding = 0,
    Callback = function(v)
        config.WeatherStormMin = v
        pcall(function() if Weather and Weather.setStormMin then Weather.setStormMin(v) end end)
    end
})
WeatherBox:AddToggle("WeatherMeteors", {
    Text = "Meteors", Default = false,
    Callback = function(v)
        config.WeatherMeteors = v
        pcall(function() if Weather and Weather.toggleMeteors then Weather.toggleMeteors(v) end end)
    end
})
WeatherBox:AddSlider("WeatherMeteorRate", {
    Text = "Meteor rate", Default = 1, Min = 0.25, Max = 3, Rounding = 2,
    Callback = function(v)
        config.WeatherMeteorRate = v
        pcall(function() if Weather and Weather.setMeteorRate then Weather.setMeteorRate(v) end end)
    end
})
WeatherBox:AddToggle("WeatherShootingStars", {
    Text = "Shooting stars", Default = false,
    Callback = function(v)
        config.WeatherShootingStars = v
        pcall(function() if Weather and Weather.toggleShootingStars then Weather.toggleShootingStars(v) end end)
    end
})
WeatherBox:AddSlider("WeatherStarRate", {
    Text = "Star rate", Default = 1, Min = 0.25, Max = 3, Rounding = 2,
    Callback = function(v)
        config.WeatherStarRate = v
        pcall(function() if Weather and Weather.setStarRate then Weather.setStarRate(v) end end)
    end
})
WeatherBox:AddToggle("WeatherPuddles", {
    Text = "Puddles", Default = false,
    Callback = function(v)
        config.WeatherPuddles = v
        pcall(function() if Weather and Weather.togglePuddles then Weather.togglePuddles(v) end end)
    end
})
WeatherBox:AddToggle("WeatherMood", {
    Text = "Mood lighting", Default = true,
    Callback = function(v)
        config.WeatherMood = v
        pcall(function() if Weather and Weather.toggleMood then Weather.toggleMood(v) end end)
    end
})
WeatherBox:AddToggle("WeatherClockDial", {
    Text = "Clock dial", Default = false,
    Callback = function(v)
        config.WeatherClockDial = v
        pcall(function() if Weather and Weather.toggleClock then Weather.toggleClock(v) end end)
    end
})
WeatherBox:AddToggle("WeatherRainbow", {
    Text = "Rainbow", Default = false,
    Callback = function(v)
        config.WeatherRainbow = v
        pcall(function() if Weather and Weather.toggleRainbow then Weather.toggleRainbow(v) end end)
    end
})
WeatherBox:AddToggle("WeatherGodRays", {
    Text = "God rays", Default = false,
    Callback = function(v)
        config.WeatherGodRays = v
        pcall(function() if Weather and Weather.toggleGodRays then Weather.toggleGodRays(v) end end)
    end
})
WeatherBox:AddDropdown("SkyboxPreset", {
    Text = "Skybox preset",
    Values = {"Off", "Space", "Sunset", "Clouds", "Storm", "Winter", "Vaporwave"},
    Default = 1,
    Callback = function(v)
        config.SkyboxPreset = v
        pcall(function() if Weather and Weather.setSkybox then Weather.setSkybox(v) end end)
    end
})




local CrossBox = Tabs.Visual:AddRightGroupbox("crosshair / mira")
CrossBox:AddToggle("Crosshair", {
    Text = "Crosshair", Default = false,
    Callback = function(v)
        config.Crosshair = v
        if v then StartCrosshair() else StopCrosshair() end
    end
})
pcall(function()
    if addColor3 then
        addColor3(CrossBox, "CrosshairColorPick", "Crosshair Color",
            function() return config.CrosshairColor or Color3.fromRGB(255, 0, 0) end,
            function(c) config.CrosshairColor = c end, function() end)
    end
end)
CrossBox:AddToggle("CrosshairSpin", {
    Text = "Spin", Default = true,
    Callback = function(v) config.CrosshairSpin = v end
})
CrossBox:AddSlider("CrosshairSpinSpeed", {
    Text = "Spin speed", Default = 120, Min = 20, Max = 360, Rounding = 0,
    Callback = function(v) config.CrosshairSpinSpeed = v end
})
CrossBox:AddSlider("CrosshairSize", {
    Text = "Line length", Default = 25, Min = 8, Max = 50, Rounding = 0,
    Callback = function(v) config.CrosshairSize = v if config.Crosshair then pcall(function() ShakoCross.build() end) end end
})
CrossBox:AddSlider("CrosshairGap", {
    Text = "Gap", Default = 10, Min = 0, Max = 30, Rounding = 0,
    Callback = function(v) config.CrosshairGap = v if config.Crosshair then pcall(function() ShakoCross.build() end) end end
})
CrossBox:AddSlider("CrosshairThickness", {
    Text = "Thickness", Default = 2, Min = 1, Max = 6, Rounding = 0,
    Callback = function(v) config.CrosshairThickness = v if config.Crosshair then pcall(function() ShakoCross.build() end) end end
})
CrossBox:AddToggle("CrosshairOutline", {
    Text = "Outline", Default = true,
    Callback = function(v) config.CrosshairOutline = v if config.Crosshair then pcall(function() ShakoCross.build() end) end end
})
CrossBox:AddToggle("CrosshairRainbow", {
    Text = "Rainbow / pulse", Default = true,
    Callback = function(v) config.CrosshairRainbow = v end
})
CrossBox:AddInput("CrosshairText", {
    Text = "Center text",
    Default = "",
    Callback = function(v) config.CrosshairText = v end
})
CrossBox:AddSlider("CrosshairTextSize", {
    Text = "Text size", Default = 22, Min = 12, Max = 36, Rounding = 0,
    Callback = function(v) config.CrosshairTextSize = v end
})
CrossBox:AddSlider("CrosshairOffsetX", {
    Text = "Offset X", Default = 0, Min = -80, Max = 80, Rounding = 0,
    Callback = function(v) config.CrosshairOffsetX = v end
})
CrossBox:AddSlider("CrosshairOffsetY", {
    Text = "Offset Y", Default = 0, Min = -80, Max = 80, Rounding = 0,
    Callback = function(v) config.CrosshairOffsetY = v end
})
CrossBox:AddSlider("CrosshairTextOffsetY", {
    Text = "Text offset Y", Default = 28, Min = 0, Max = 80, Rounding = 0,
    Callback = function(v) config.CrosshairTextOffsetY = v end
})
CrossBox:AddLabel("Alinea con la mira real de Rivals")

local HitFXBox = Tabs.Visual:AddLeftGroupbox("hit feedback")
HitFXBox:AddToggle("FXHitMarker", {
    Text = "Hit marker", Default = false,
    Callback = function(v) config.FXHitMarker = v end
})
addColor3(HitFXBox, "HitMarkerColorPick", "Hit Marker Color",
    function() return config.FXHitMarkerColor or Color3.fromRGB(255,255,255) end,
    function(c) config.FXHitMarkerColor = c end, function() end)
addColor3(HitFXBox, "HitMarkerCritPick", "Hit Crit Color",
    function() return config.FXHitMarkerCritColor or Color3.fromRGB(255,194,75) end,
    function(c) config.FXHitMarkerCritColor = c end, function() end)

HitFXBox:AddDropdown("FXHitMarkerStyle", {
    Text = "Marker style",
    Values = {"X", "Plus", "Circle"},
    Default = 1,
    Callback = function(v) config.FXHitMarkerStyle = v end
})
HitFXBox:AddSlider("FXHitMarkerGap", {
    Text = "Marker gap", Default = 5, Min = 2, Max = 20, Rounding = 0,
    Callback = function(v) config.FXHitMarkerGap = v end
})
HitFXBox:AddSlider("FXHitMarkerLen", {
    Text = "Marker length", Default = 8, Min = 2, Max = 24, Rounding = 0,
    Callback = function(v) config.FXHitMarkerLen = v end
})
HitFXBox:AddSlider("FXHitMarkerThickness", {
    Text = "Marker thickness", Default = 2, Min = 1, Max = 6, Rounding = 0,
    Callback = function(v) config.FXHitMarkerThickness = v end
})
HitFXBox:AddToggle("FXBeamTracer", {
    Text = "Bullet tracer", Default = false,
    Callback = function(v) config.FXBeamTracer = v end
})
HitFXBox:AddDropdown("FXBeamStyle", {
    Text = "Style",
    Values = {"Line", "Glow", "Prism", "Arc"},
    Default = 1,
    Callback = function(v) config.FXBeamStyle = v end
})
HitFXBox:AddSlider("FXBeamDuration", {
    Text = "Tracer duration", Default = 0.35, Min = 0.1, Max = 2, Rounding = 2,
    Callback = function(v) config.FXBeamDuration = v end
})
HitFXBox:AddSlider("FXBeamThickness", {
    Text = "Tracer thickness", Default = 0.12, Min = 0.04, Max = 0.5, Rounding = 2,
    Callback = function(v) config.FXBeamThickness = v end
})
HitFXBox:AddToggle("FXDamageNumbers", {
    Text = "Damage numbers", Default = false,
    Callback = function(v) config.FXDamageNumbers = v end
})
HitFXBox:AddToggle("FXKillBanner", {
    Text = "Kill banner", Default = false,
    Callback = function(v) config.FXKillBanner = v end
})
HitFXBox:AddLabel("LuaHook-style · white hit / gold crit / red lethal")


-- ==================== AMBIENT SOUNDS (relajantes) ====================
-- IDs públicos que suelen cargar (si uno falla, se prueba el fallback)
local AMBIENT_SOUNDS = {
    ["Rain"]         = { "rbxassetid://1516791621", "rbxassetid://5114135799", "rbxassetid://6066411733" },
    ["Heavy Rain"]   = { "rbxassetid://5114135799", "rbxassetid://1516791621", "rbxassetid://6066411733" },
    ["Thunder"]      = { "rbxassetid://1516791621", "rbxassetid://5114135799" },
    ["Forest"]       = { "rbxassetid://3472701279", "rbxassetid://3422304783", "rbxassetid://224851262" },
    ["Birds"]        = { "rbxassetid://3472701279", "rbxassetid://3422304783" },
    ["Wind"]         = { "rbxassetid://224851262", "rbxassetid://336142745", "rbxassetid://5987864649" },
    ["Ocean"]        = { "rbxassetid://166475216", "rbxassetid://4792901531", "rbxassetid://5987864649" },
    ["Campfire"]     = { "rbxassetid://317656970", "rbxassetid://4601689861" },
    ["Night"]        = { "rbxassetid://3458023152", "rbxassetid://3472701279", "rbxassetid://5987864649" },
    ["River"]        = { "rbxassetid://166475216", "rbxassetid://4792901531" },
    ["Soft Piano"]   = { "rbxassetid://1845554019", "rbxassetid://1837829205" },
    ["LoFi"]         = { "rbxassetid://9043887091", "rbxassetid://1845554019" },
    ["Rain Classic"] = { "rbxassetid://1516791621" },
    ["Wind Classic"] = { "rbxassetid://224851262" },
    ["Fire Classic"] = { "rbxassetid://317656970" },
}

local ambientSoundObj = nil
local ambientToken = 0

local function stopAmbientSound()
    ambientToken = ambientToken + 1 -- cancela waits pendientes
    config.AmbientSoundEnabled = false
    -- parar el objeto actual
    if ambientSoundObj then
        pcall(function() ambientSoundObj:Stop() end)
        pcall(function() ambientSoundObj.Volume = 0 end)
        pcall(function() ambientSoundObj:Destroy() end)
        ambientSoundObj = nil
    end
    -- limpiar TODOS los ShakoAmbient (por si quedaron clones)
    pcall(function()
        for _, parent in ipairs({ SoundService, workspace, LocalPlayer:FindFirstChild("PlayerGui") }) do
            if not parent then continue end
            for _, s in ipairs(parent:GetDescendants()) do
                if s:IsA("Sound") and (s.Name == "ShakoAmbient" or s.Name:find("ShakoAmbient", 1, true)) then
                    pcall(function() s:Stop() end)
                    pcall(function() s.Volume = 0 end)
                    pcall(function() s:Destroy() end)
                end
            end
            for _, s in ipairs(parent:GetChildren()) do
                if s:IsA("Sound") and s.Name == "ShakoAmbient" then
                    pcall(function() s:Stop() end)
                    pcall(function() s:Destroy() end)
                end
            end
        end
    end)
    -- también en Character HRP
    pcall(function()
        local char = LocalPlayer.Character
        if char then
            for _, s in ipairs(char:GetDescendants()) do
                if s:IsA("Sound") and s.Name == "ShakoAmbient" then
                    pcall(function() s:Stop() s:Destroy() end)
                end
            end
        end
    end)
end

local function playAmbientSound()
    -- stop limpio sin apagar el flag (play lo vuelve a poner)
    local wasOn = true
    local tok = ambientToken + 1
    ambientToken = tok
    if ambientSoundObj then
        pcall(function() ambientSoundObj:Stop() end)
        pcall(function() ambientSoundObj:Destroy() end)
        ambientSoundObj = nil
    end
    pcall(function()
        for _, s in ipairs(SoundService:GetChildren()) do
            if s:IsA("Sound") and s.Name == "ShakoAmbient" then
                pcall(function() s:Stop() s:Destroy() end)
            end
        end
    end)

    if not config.AmbientSoundEnabled then return end

    local name = config.AmbientSoundType or "Rain"
    local list = AMBIENT_SOUNDS[name] or AMBIENT_SOUNDS["Rain"]
    if type(list) == "string" then list = { list } end

    local vol = math.clamp(tonumber(config.AmbientSoundVolume) or 0.45, 0, 2)
    local pitch = math.clamp(tonumber(config.AmbientSoundPitch) or 1, 0.5, 1.5)

    local function tryId(uri)
        if ambientToken ~= tok then return false end
        local s = Instance.new("Sound")
        s.Name = "ShakoAmbient"
        s.SoundId = uri
        s.Looped = true
        s.Volume = vol
        s.PlaybackSpeed = pitch
        s.PlayOnRemove = false
        pcall(function()
            s.RollOffMode = Enum.RollOffMode.Linear
            s.RollOffMaxDistance = 100000
            s.RollOffMinDistance = 10000
        end)
        s.Parent = SoundService
        -- NO usar PlayLocalSound (crea instancia que no se puede stoppear fácil)
        local okPlay = pcall(function() s:Play() end)
        ambientSoundObj = s

        -- esperar carga
        local t0 = tick()
        while ambientToken == tok and s.Parent and not s.IsLoaded and (tick() - t0) < 2.5 do
            task.wait(0.05)
        end
        if ambientToken ~= tok then
            pcall(function() s:Stop() s:Destroy() end)
            return false
        end
        -- si cargó o está sonando
        if s.IsLoaded or s.IsPlaying or (s.TimeLength and s.TimeLength > 0) then
            if not s.IsPlaying then pcall(function() s:Play() end) end
            return true
        end
        -- falló este id
        pcall(function() s:Stop() s:Destroy() end)
        if ambientSoundObj == s then ambientSoundObj = nil end
        return false
    end

    task.spawn(function()
        for _, uri in ipairs(list) do
            if ambientToken ~= tok then return end
            if tryId(uri) then
                return
            end
        end
        -- último recurso: rain clásico
        if ambientToken == tok and config.AmbientSoundEnabled then
            tryId("rbxassetid://1516791621")
        end
    end)
end

function StartAmbientSound()
    config.AmbientSoundEnabled = true
    playAmbientSound()
end

function StopAmbientSound()
    stopAmbientSound()
    Notify("Ambient", "OFF")
end

function SetAmbientVolume(v)
    config.AmbientSoundVolume = v
    if ambientSoundObj and ambientSoundObj.Parent then
        pcall(function() ambientSoundObj.Volume = math.clamp(tonumber(v) or 0.45, 0, 2) end)
    end
end

local AmbientBox = Tabs.World:AddLeftGroupbox("Ambient")
AmbientBox:AddToggle("AmbientSoundEnabled", {
    Text = "Ambient Sound", Default = false,
    Callback = function(v)
        if v then
            StartAmbientSound()
            Notify("Ambient", "ON · " .. tostring(config.AmbientSoundType or "Rain"))
        else
            StopAmbientSound()
        end
    end
})
AmbientBox:AddDropdown("AmbientSoundType", {
    Text = "Tipo",
    Values = {"Rain", "Heavy Rain", "Thunder", "Forest", "Birds", "Wind", "Ocean", "Campfire", "Night", "River", "Soft Piano", "LoFi", "Rain Classic", "Wind Classic", "Fire Classic"},
    Default = 1,
    Callback = function(v)
        config.AmbientSoundType = v
        if config.AmbientSoundEnabled then
            playAmbientSound()
        end
    end
})
AmbientBox:AddSlider("AmbientSoundVolume", {
    Text = "Volumen", Default = 0.45, Min = 0.05, Max = 1.5, Rounding = 2,
    Callback = function(v) SetAmbientVolume(v) end
})
AmbientBox:AddSlider("AmbientSoundPitch", {
    Text = "Pitch", Default = 1, Min = 0.6, Max = 1.4, Rounding = 2,
    Callback = function(v)
        config.AmbientSoundPitch = v
        if ambientSoundObj and ambientSoundObj.Parent then
            pcall(function() ambientSoundObj.PlaybackSpeed = v end)
        end
    end
})
AmbientBox:AddButton({
    Text = "Play / Restart",
    Func = function()
        config.AmbientSoundEnabled = true
        playAmbientSound()
        Notify("Ambient", tostring(config.AmbientSoundType or "Rain"))
    end
})
AmbientBox:AddButton({
    Text = "Stop",
    Func = function()
        StopAmbientSound()
    end
})
AmbientBox:AddLabel("Stop ahora sí corta · varios IDs por tipo")


local EmoteBox = Tabs.Character:AddLeftGroupbox("emotes")
EmoteBox:AddDropdown("EmotePreset", {
    Text = "Preset",
    Values = {"Floss", "Orange Justice", "Take The L", "Default Dance", "Electro Shuffle", "Fresh", "Robot", "Hype", "Dab", "Laugh", "Wave", "Gangnam", "Shuffle", "Breakdance", "Twist", "Zombie", "Custom A", "Custom B", "Custom C", "Custom D", "Custom E", "Custom F", "Custom G", "Custom H", "Custom I", "Custom J", "Custom K", "Custom L", "Custom"},
    Default = 1,
    Callback = function(v)
        config.EmoteName = v
        if v ~= "Custom" and EMOTES and EMOTES[v] then
            config.EmoteId = tostring(EMOTES[v])
        end
        -- al elegir, se reproduce y queda activa
        if v ~= "Custom" then
            task.defer(function()
                PlayEmote(config.EmoteId, config.EmoteSpeed)
            end)
        end
    end
})
EmoteBox:AddInput("EmoteIdInput", {
    Text = "Animation ID (custom)",
    Default = "5917459365",
    Finished = true,
    Callback = function(v)
        config.EmoteId = tostring(v):gsub("%D", "")
        config.EmoteName = "Custom"
        if config.EmoteId ~= "" then
            task.defer(function() PlayEmote(config.EmoteId, config.EmoteSpeed) end)
        end
    end
})
EmoteBox:AddSlider("EmoteSpeed", {
    Text = "Velocidad emote", Default = 1, Min = 0.1, Max = 10, Rounding = 1,
    Callback = function(v)
        config.EmoteSpeed = tonumber(v) or 1
        -- aplica al instante al track actual
        pcall(function()
            if currentAnimTrack then
                currentAnimTrack:AdjustSpeed(config.EmoteSpeed)
                -- algunos tracks ignoran AdjustSpeed si no están playing
                if currentAnimTrack.IsPlaying then
                    currentAnimTrack:Play(0, 1, config.EmoteSpeed)
                end
            end
        end)
    end
})
EmoteBox:AddToggle("EmoteAlwaysOn", {
    Text = "Always on (auto loop)", Default = true,
    Callback = function(v)
        config.EmoteKeepPlaying = v
        config.EmoteLoop = true
        if v and config.EmoteId then
            PlayEmote(config.EmoteId, config.EmoteSpeed)
        elseif not v then
            StopEmote()
        end
    end
})
EmoteBox:AddButton({
    Text = "Play / Restart",
    Func = function()
        local id = config.EmoteId
        if (not id or id == "") and config.EmoteName and EMOTES[config.EmoteName] then
            id = EMOTES[config.EmoteName]
        end
        PlayEmote(id, config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Stop Emote",
    Func = function() StopEmote() end
})
EmoteBox:AddLabel("Siempre loop · se mantiene al caminar/respawn")
EmoteBox:AddButton({
    Text = "Play Custom A (86329...)",
    Func = function()
        config.EmoteId = "86329197824055"
        config.EmoteName = "Custom A"
        PlayEmote("86329197824055", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom B (91729...)",
    Func = function()
        config.EmoteId = "91729309021707"
        config.EmoteName = "Custom B"
        PlayEmote("91729309021707", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom C (133811...)",
    Func = function()
        config.EmoteId = "133811691098518"
        config.EmoteName = "Custom C"
        PlayEmote("133811691098518", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom D (133375...)",
    Func = function()
        config.EmoteId = "133375187065498"
        config.EmoteName = "Custom D"
        PlayEmote("133375187065498", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom E (130601...)",
    Func = function()
        config.EmoteId = "130601328561804"
        config.EmoteName = "Custom E"
        PlayEmote("130601328561804", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom F (129571...)",
    Func = function()
        config.EmoteId = "129571341119376"
        config.EmoteName = "Custom F"
        PlayEmote("129571341119376", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom G (79902...)",
    Func = function()
        config.EmoteId = "79902725961872"
        config.EmoteName = "Custom G"
        PlayEmote("79902725961872", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom H (119060...)",
    Func = function()
        config.EmoteId = "119060168059843"
        config.EmoteName = "Custom H"
        PlayEmote("119060168059843", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom I (82995...)",
    Func = function()
        config.EmoteId = "82995540773684"
        config.EmoteName = "Custom I"
        PlayEmote("82995540773684", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom J (78224...)",
    Func = function()
        config.EmoteId = "78224683906191"
        config.EmoteName = "Custom J"
        PlayEmote("78224683906191", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom K (73562...)",
    Func = function()
        config.EmoteId = "73562814360939"
        config.EmoteName = "Custom K"
        PlayEmote("73562814360939", config.EmoteSpeed)
    end
})
EmoteBox:AddButton({
    Text = "Play Custom L (88425...)",
    Func = function()
        config.EmoteId = "88425531063616"
        config.EmoteName = "Custom L"
        PlayEmote("88425531063616", config.EmoteSpeed)
    end
})



local AUBox = Tabs.Misc:AddLeftGroupbox("ue counter")
local togUE = AUBox:AddToggle("UECounterToggle", {
    Text = "UE Counter", Default = false,
    Callback = function(v)
        config.UECounterEnabled = v
        config.AntiUnnamed = v
        if v then
            StartUECounter()
            StartAUWatcher()
        else
            StopUECounter()
        end
    end
})
bindKey(togUE, "UECounterKey", "...")

AUBox:AddSlider("UECounterY", {
    Text = "Y distance", Default = 500, Min = 100, Max = 1000, Rounding = 0,
    Callback = function(v) config.UECounterY = v end
})
AUBox:AddSlider("UECounterCooldown", {
    Text = "Teleport cooldown", Default = 0.2, Min = 0, Max = 1, Rounding = 2,
    Callback = function(v) config.UECounterCooldown = v end
})


local DefBox = Tabs.Misc:AddLeftGroupbox("defense")
DefBox:AddToggle("HurtEvade", {
    Text = "Hurt Evade (al recibir dmg)", Default = false,
    Callback = function(v)
        config.HurtEvade = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("HurtEvadeDist", {
    Text = "Hurt Evade Dist", Default = 18, Min = 5, Max = 150, Rounding = 0,
    Callback = function(v) config.HurtEvadeDist = v end
})
DefBox:AddToggle("NearEnemyEvade", {
    Text = "Evade si enemy cerca", Default = false,
    Callback = function(v)
        config.NearEnemyEvade = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("NearEnemyDist", {
    Text = "Near Dist", Default = 8, Min = 3, Max = 25, Rounding = 0,
    Callback = function(v) config.NearEnemyDist = v end
})
DefBox:AddToggle("AntiMeleeEnabled", {
    Text = "Anti Melee / body", Default = true,
    Callback = function(v) config.AntiMeleeEnabled = v end
})
DefBox:AddSlider("AntiMeleeRange", {
    Text = "Anti Melee Range", Default = 7, Min = 3, Max = 80, Rounding = 0,
    Callback = function(v) config.AntiMeleeRange = v end
})
DefBox:AddToggle("AntiUEClose", {
    Text = "Anti UE Close (push lejos)", Default = false,
    Callback = function(v)
        config.AntiUEClose = v
        if v then StartDefense() end
    end
})
DefBox:AddSlider("AntiUECloseDist", {
    Text = "UE Close Dist", Default = 35, Min = 10, Max = 120, Rounding = 0,
    Callback = function(v) config.AntiUECloseDist = v end
})
DefBox:AddSlider("AntiUEClosePush", {
    Text = "UE Close Push", Default = 80, Min = 20, Max = 200, Rounding = 0,
    Callback = function(v) config.AntiUEClosePush = v end
})
DefBox:AddToggle("AntiUEPeek", {
    Text = "Anti UE Peek (side hop)", Default = false,
    Callback = function(v)
        config.AntiUEPeek = v
        if v then StartDefense() end
    end
})
DefBox:AddToggle("DesyncSpam", {
    Text = "Desync Spam (anti hit)", Default = false,
    Callback = function(v)
        config.DesyncSpam = v
        if v then StartDefense() end
    end
})
DefBox:AddSlider("DesyncSpamPower", {
    Text = "Desync Spam Power", Default = 8, Min = 2, Max = 20, Rounding = 0,
    Callback = function(v) config.DesyncSpamPower = v end
})
DefBox:AddToggle("StickFarMode", {
    Text = "Stick LEJOS (anti melee UE)", Default = false,
    Callback = function(v) config.StickFarMode = v end
})

DefBox:AddToggle("AdaptiveDesync", {
    Text = "Adaptive Desync (micro move)", Default = false,
    Callback = function(v)
        config.AdaptiveDesync = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
        Notify("Defense", v and "Adaptive ON" or "Adaptive OFF")
    end
})
DefBox:AddSlider("AdaptiveDesyncPower", {
    Text = "Desync Power", Default = 6, Min = 1, Max = 20, Rounding = 0,
    Callback = function(v) config.AdaptiveDesyncPower = v end
})
DefBox:AddToggle("VelocityBreak", {
    Text = "Velocity Break", Default = true,
    Callback = function(v) config.VelocityBreak = v end
})
DefBox:AddSlider("VelocityBreakMax", {
    Text = "Max Velocity", Default = 140, Min = 60, Max = 300, Rounding = 0,
    Callback = function(v) config.VelocityBreakMax = v end
})
DefBox:AddToggle("LowHPPanic", {
    Text = "Low HP Panic", Default = true,
    Callback = function(v) config.LowHPPanic = v end
})
DefBox:AddSlider("LowHPThreshold", {
    Text = "Low HP %", Default = 30, Min = 5, Max = 80, Rounding = 0,
    Callback = function(v) config.LowHPThreshold = v end
})
DefBox:AddToggle("RandomMicroMove", {
    Text = "Random Micro Move", Default = false,
    Callback = function(v)
        config.RandomMicroMove = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddDivider()
DefBox:AddLabel("—— Anti-Rager ——")
DefBox:AddToggle("RagerCombo", {
    Text = "Rager Combo (preset)", Default = false,
    Callback = function(v)
        config.RagerCombo = v
        if v then
            config.OrbitEvade = true
            config.HeightJitter = true
            config.AntiHeadTP = true
            config.StrafePattern = true
            config.VelocityBreak = true
            config.DesyncSpam = true
            config.HurtEvade = true
            StartDefense()
            Notify("HVH", "Rager Combo ON")
        else
            if anyDefenseOn() then StartDefense() else StopDefense() end
            Notify("HVH", "Rager Combo OFF")
        end
    end
})
DefBox:AddToggle("OrbitEvade", {
    Text = "Orbit Evade (rompe head-TP)", Default = false,
    Callback = function(v)
        config.OrbitEvade = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("OrbitEvadeRadius", {
    Text = "Orbit Radius", Default = 12, Min = 4, Max = 40, Rounding = 0,
    Callback = function(v) config.OrbitEvadeRadius = v end
})
DefBox:AddSlider("OrbitEvadeSpeed", {
    Text = "Orbit Speed", Default = 8, Min = 2, Max = 25, Rounding = 0,
    Callback = function(v) config.OrbitEvadeSpeed = v end
})
DefBox:AddToggle("HeightJitter", {
    Text = "Height Jitter (anti aim)", Default = false,
    Callback = function(v)
        config.HeightJitter = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("HeightJitterAmount", {
    Text = "Height Amount", Default = 6, Min = 1, Max = 20, Rounding = 0,
    Callback = function(v) config.HeightJitterAmount = v end
})
DefBox:AddToggle("AntiHeadTP", {
    Text = "Anti Head-TP (si están arriba)", Default = false,
    Callback = function(v)
        config.AntiHeadTP = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("AntiHeadTPPush", {
    Text = "Head-TP Push", Default = 35, Min = 10, Max = 100, Rounding = 0,
    Callback = function(v) config.AntiHeadTPPush = v end
})
DefBox:AddToggle("StrafePattern", {
    Text = "Strafe Pattern (A-D)", Default = false,
    Callback = function(v)
        config.StrafePattern = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("StrafePatternDist", {
    Text = "Strafe Dist", Default = 10, Min = 3, Max = 30, Rounding = 0,
    Callback = function(v) config.StrafePatternDist = v end
})
DefBox:AddToggle("FakePeek", {
    Text = "Fake Peek (lado y vuelve)", Default = false,
    Callback = function(v)
        config.FakePeek = v
        if anyDefenseOn() then StartDefense() else StopDefense() end
    end
})
DefBox:AddSlider("FakePeekDist", {
    Text = "Peek Dist", Default = 18, Min = 5, Max = 50, Rounding = 0,
    Callback = function(v) config.FakePeekDist = v end
})
DefBox:AddLabel("Combo + Orbit + Height = mejor vs ragers")

local AABox = Tabs.Character:AddRightGroupbox("anti-aim")
AABox:AddToggle("AntiAimToggle", {
    Text = "Anti Aim", Default = false,
    Callback = function(v) config.AntiAimEnabled = v if v then StartAntiAim() else StopAntiAim() end end
})
AABox:AddDropdown("AntiAimMode", {
    Text = "Modo",
    Values = {"Underground", "Spin", "Jitter", "Random", "Backwards", "Sideways", "ManualYaw"},
    Default = 1,
    Callback = function(v) config.AntiAimMode = v end
})
AABox:AddSlider("UndergroundPitch", {
    Text = "Underground Pitch", Default = -25, Min = -60, Max = -5, Rounding = 0,
    Callback = function(v)
        config.UndergroundPitch = v
        config.AntiAimUndergroundY = v -- compat
    end
})
AABox:AddSlider("UndergroundJitter", {
    Text = "Underground Jitter", Default = 35, Min = 0, Max = 90, Rounding = 0,
    Callback = function(v) config.UndergroundJitter = v end
})
AABox:AddSlider("UndergroundSpin", {
    Text = "Underground Spin Factor", Default = 0.4, Min = 0, Max = 2, Rounding = 1,
    Callback = function(v) config.UndergroundSpin = v end
})
AABox:AddToggle("UndergroundRandom", {
    Text = "Underground Randomize", Default = true,
    Callback = function(v) config.UndergroundRandom = v end
})
AABox:AddLabel("Underground = mira al piso + jitter (no TP)")
AABox:AddLabel("Funciona CON rage (otros lo ven, vos no)")
AABox:AddSlider("SpinSpeed", {
    Text = "Spin", Default = 25, Min = 5, Max = 100, Rounding = 0,
    Callback = function(v) config.SpinSpeed = v end
})
AABox:AddSlider("JitterAngle", {
    Text = "Jitter", Default = 45, Min = 10, Max = 90, Rounding = 0,
    Callback = function(v) config.JitterAngle = v end
})
AABox:AddSlider("ManualYaw", {
    Text = "Manual Yaw", Default = 180, Min = 0, Max = 360, Rounding = 0,
    Callback = function(v) config.ManualYaw = v end
})
AABox:AddSlider("AAPitch", {
    Text = "Pitch", Default = 0, Min = -89, Max = 89, Rounding = 0,
    Callback = function(v) config.AAPitch = v end
})
AABox:AddToggle("AAInvert", {
    Text = "Invert", Default = false,
    Callback = function(v) config.AAInvert = v end
})

local MoveBox = Tabs.Character:AddLeftGroupbox("movement")
local togFly = MoveBox:AddToggle("FlyToggle", {
    Text = "Fly", Default = false,
    Callback = function(v) config.FlyEnabled = v if v then StartFly() else StopFly() end end
})
bindKey(togFly, "FlyKey", "...")

MoveBox:AddSlider("FlySpeed", {
    Text = "Fly Speed", Default = 120, Min = 30, Max = 9999, Rounding = 0,
    Callback = function(v) config.FlySpeed = v end
})
MoveBox:AddSlider("FlyBoost", {
    Text = "Fly Boost (Shift)", Default = 1.8, Min = 1, Max = 5, Rounding = 1,
    Callback = function(v) config.FlyBoost = v end
})
local togNoclip = MoveBox:AddToggle("NoclipToggle", {
    Text = "Noclip", Default = false,
    Callback = function(v) config.NoclipEnabled = v if v then StartNoclip() else StopNoclip() end end
})
bindKey(togNoclip, "NoclipKey", "...")

MoveBox:AddToggle("SpeedToggle", {
    Text = "Speed (Rivals)", Default = false,
    Callback = function(v)
        config.SpeedEnabled = v
        if v then
            ApplySpeedState()
            Notify("Speed", "ON " .. tostring(config.WalkSpeed))
        else
            ApplySpeedState() -- restaura WalkSpeed default
            Notify("Speed", "OFF · normal")
        end
    end
})
MoveBox:AddToggle("SpeedVelocityMode", {
    Text = "Speed Velocity Mode (Rivals)", Default = true,
    Callback = function(v) config.SpeedVelocityMode = v end
})
MoveBox:AddSlider("WalkSpeed", {
    Text = "WalkSpeed", Default = 50, Min = 16, Max = 500, Rounding = 0,
    Callback = function(v)
        config.WalkSpeed = v
        if config.SpeedEnabled then ApplySpeedState() end
    end
})
MoveBox:AddToggle("JumpPowerToggle", {
    Text = "Jump Power (Rivals)", Default = false,
    Callback = function(v)
        config.JumpPowerEnabled = v
        ApplyJumpState()
        Notify("Jump", v and ("ON " .. tostring(config.JumpPower)) or "OFF · normal")
    end
})
MoveBox:AddToggle("JumpBoostForce", {
    Text = "Jump impulso Y (Rivals)", Default = true,
    Callback = function(v) config.JumpBoostForce = v end
})
MoveBox:AddSlider("JumpPower", {
    Text = "JumpPower", Default = 100, Min = 50, Max = 9999, Rounding = 0,
    Callback = function(v) config.JumpPower = v end
})
MoveBox:AddToggle("SlideBoost", {
    Text = "Slide Boost", Default = false,
    Callback = function(v)
        config.SlideBoostEnabled = v
        if v then StartSlideBoostLoop() else StopSlideBoostLoop() end
        Notify("Slide", v and ("ON x" .. tostring(config.SlideBoostMult)) or "OFF")
    end
})
MoveBox:AddSlider("SlideBoostMult", {
    Text = "Slide Boost Mult", Default = 2.5, Min = 1.2, Max = 12, Rounding = 1,
    Callback = function(v) config.SlideBoostMult = v end
})
MoveBox:AddSlider("SlideBaseSpeed", {
    Text = "Slide Base Speed", Default = 45, Min = 20, Max = 200, Rounding = 0,
    Callback = function(v) config.SlideBaseSpeed = v end
})
MoveBox:AddDropdown("SlideBoostKey", {
    Text = "Slide Key",
    Values = {"C", "LeftControl", "LeftShift", "Q", "V"},
    Default = 1,
    Callback = function(v) config.SlideBoostKey = v end
})
MoveBox:AddToggle("InfinityJump", {
    Text = "Infinite Jump", Default = false,
    Callback = function(v) config.InfinityJump = v end
})
MoveBox:AddToggle("AntiFling", {
    Text = "Anti Fling", Default = true,
    Callback = function(v) config.AntiFling = v end
})
MoveBox:AddLabel("Rivals pisa WalkSpeed: usa Velocity Mode")
MoveBox:AddLabel("Slide: mantén la tecla + WASD")


-- ==================== SPOOFER + UNLOCK UI (AURORA) ====================
pcall(function()
    if not Tabs.Spoofer then return end

    local PlatBox = Tabs.Spoofer:AddLeftGroupbox("Platform / Device")
    PlatBox:AddDropdown("RivalsDeviceType", {
        Text = "Device",
        Values = {"Desktop", "Mobile", "Console", "VR"},
        Default = 1,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.device = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.device = v end
        end
    })
    PlatBox:AddToggle("RivalsSpoofDevice", {
        Text = "Spoof Device", Default = false,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.spoof_device = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.spoof_device = v end
            RivalsApplyDevice()
        end
    })
    PlatBox:AddToggle("RivalsDeviceSpam", {
        Text = "Device Spam", Default = false,
        Callback = function(v) RivalsDeviceSpam(v) end
    })
    PlatBox:AddSlider("RivalsSpamRate", {
        Text = "Spam Rate (s)", Default = 1, Min = 0.2, Max = 5, Rounding = 1,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.spam_rate = v end
        end
    })
    PlatBox:AddButton({ Text = "Apply Device", Func = function() RivalsApplyDevice() end })

    local NameBoxR = Tabs.Spoofer:AddLeftGroupbox("Name Spoof")
    NameBoxR:AddInput("RivalsDispName", {
        Text = "Display Name", Default = LocalPlayer.DisplayName,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.display_name_value = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.display_name_value = v end
            if RivalsSpoof and RivalsSpoof.display_name then RivalsRefreshNames() end
        end
    })
    NameBoxR:AddToggle("RivalsSpoofDisp", {
        Text = "Spoof Display Name", Default = false,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.display_name = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.display_name = v end
            if config then
                config.SpoofEnabled = v and true or config.SpoofEnabled
                config.SpoofMyName = v
                config.SpoofProfileUI = true
                if st then config.SpoofMyNameValue = tostring(st.display_name_value or "") end
            end
            RivalsRefreshNames()
            task.delay(0.2, RivalsRefreshNames)
            Notify("Name", v and "Display spoof ON (ESC + killfeed)" or "OFF")
        end
    })
    NameBoxR:AddInput("RivalsUserName", {
        Text = "Username", Default = LocalPlayer.Name,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.username_value = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.username_value = v end
        end
    })
    NameBoxR:AddToggle("RivalsSpoofUser", {
        Text = "Spoof Username", Default = false,
        Callback = function(v)
            if RivalsSpoof then RivalsSpoof.username = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.username = v end
            RivalsRefreshNames()
            task.delay(0.2, RivalsRefreshNames)
        end
    })
    NameBoxR:AddButton({
        Text = "Aplicar Name (ESC + killfeed)",
        Func = function()
            RivalsRefreshNames()
            task.delay(0.25, RivalsRefreshNames)
            Notify("Name", "Refrescado — abre ESC para ver lista")
        end
    })

    local AvBox = Tabs.Spoofer:AddLeftGroupbox("Avatar / Badges")
    AvBox:AddInput("VictimUserId", {
        Text = "Avatar UserId",
        Default = tostring(LocalPlayer.UserId),
        Numeric = true,
        Callback = function(v)
            local id = tonumber(v) or LocalPlayer.UserId
            if RivalsSpoof then RivalsSpoof.avatar_userid = id end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st then st.avatar_userid = id end
        end
    })
    AvBox:AddToggle("SpoofAvatar", {
        Text = "Spoof Avatar", Default = false,
        Callback = function(v)
            RivalsSetAvatarSpoof(v)
            if v and ShakoSync then pcall(ShakoSync.broadcast) end
        end
    })
    AvBox:AddButton({
        Text = "Apply Avatar",
        Func = function() RivalsSetAvatarSpoof(true) end
    })

    AvBox:AddToggle("ShakoSyncPeers", {
        Text = "Ver skins de otros shako.win",
        Default = true,
        Callback = function(v)
            config.ShakoSyncEnabled = v
            if ShakoSync then
                if v then ShakoSync.start() else ShakoSync.stop() end
            end
        end
    })
    AvBox:AddLabel("Si el otro usa shako + avatar spoof, lo ves cyan")

    AvBox:AddToggle("AnonymousMode", {
        Text = "Anonymous Mode", Default = false,
        Callback = function(v) RivalsSetAnonymous(v) end
    })
    AvBox:AddToggle("BadgePremium", {
        Text = "Premium Badge", Default = false,
        Callback = function(v)
            if RivalsSpoof and RivalsSpoof.badges then RivalsSpoof.badges.Premium = v end
            RivalsUpdateBadges()
        end
    })
    AvBox:AddToggle("BadgeVerified", {
        Text = "Verified Badge", Default = false,
        Callback = function(v)
            if RivalsSpoof and RivalsSpoof.badges then RivalsSpoof.badges.Verified = v end
            RivalsUpdateBadges()
        end
    })
    AvBox:AddToggle("BadgeInfluencer", {
        Text = "Influencer Badge", Default = false,
        Callback = function(v)
            if RivalsSpoof and RivalsSpoof.badges then RivalsSpoof.badges.Influencer = v end
            RivalsUpdateBadges()
        end
    })
    AvBox:AddToggle("BadgeAdmin", {
        Text = "Admin Badge", Default = false,
        Callback = function(v)
            if RivalsSpoof and RivalsSpoof.badges then RivalsSpoof.badges.Admin = v end
            RivalsUpdateBadges()
        end
    })
    AvBox:AddButton({ Text = "Apply Badges", Func = function() RivalsUpdateBadges() end })

    local LBBox = Tabs.Spoofer:AddRightGroupbox("Leaderboard Spoof")
    local function lbSet(flag, val)
        if RivalsSpoof and RivalsSpoof.lb then RivalsSpoof.lb[flag] = val end
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        if st and st.leaderboard then
            local map = {elo="ELO",streak="Streak",kills="Kills",wins="Wins",level="Level"}
            if map[flag] then st.leaderboard[map[flag]] = val
            else st.leaderboard[flag] = val end
        end
        pcall(RivalsApplyLeaderboard)
    end
    LBBox:AddToggle("LBElo", { Text = "Spoof ELO", Default = false, Callback = function(v) lbSet("elo", v) end })
    LBBox:AddSlider("LBEloVal", { Text = "ELO Value", Default = 2500, Min = 0, Max = 999999999, Rounding = 0, Callback = function(v) lbSet("elo_value", v) end })
    LBBox:AddToggle("LBStreak", { Text = "Spoof Streak", Default = false, Callback = function(v) lbSet("streak", v) end })
    LBBox:AddSlider("LBStreakVal", { Text = "Streak Value", Default = 50, Min = 0, Max = 999999999, Rounding = 0, Callback = function(v) lbSet("streak_value", v) end })
    LBBox:AddToggle("LBKills", { Text = "Spoof Kills", Default = false, Callback = function(v) lbSet("kills", v) end })
    LBBox:AddSlider("LBKillsVal", { Text = "Kills Value", Default = 9999, Min = 0, Max = 999999999, Rounding = 0, Callback = function(v) lbSet("kills_value", v) end })
    LBBox:AddToggle("LBWins", { Text = "Spoof Wins", Default = false, Callback = function(v) lbSet("wins", v) end })
    LBBox:AddSlider("LBWinsVal", { Text = "Wins Value", Default = 500, Min = 0, Max = 999999999, Rounding = 0, Callback = function(v) lbSet("wins_value", v) end })
    LBBox:AddToggle("LBLevel", { Text = "Spoof Level", Default = false, Callback = function(v) lbSet("level", v) end })
    LBBox:AddSlider("LBLevelVal", { Text = "Level Value", Default = 100, Min = 1, Max = 999999999, Rounding = 0, Callback = function(v) lbSet("level_value", v) end })
    LBBox:AddButton({ Text = "Apply Leaderboard", Func = function() RivalsApplyLeaderboard() end })

    local CurBox = Tabs.Spoofer:AddRightGroupbox("Currency Spoof")
    CurBox:AddButton({ Text = "Apply Currency", Func = function() RivalsUpdateCurrency() end })


    -- Rank Spoofer (AURORA charms + ELO map)
    kCharmRanks = {
        "Archnemesis",
        "Nemesis",
        "Onyx 3", "Onyx 2", "Onyx 1",
        "Diamond 3", "Diamond 2", "Diamond 1",
        "Platinum 3", "Platinum 2", "Platinum 1",
        "Gold 3", "Gold 2", "Gold 1",
        "Silver 3", "Silver 2", "Silver 1",
        "Bronze 3", "Bronze 2", "Bronze 1",
        "Unranked",
        "Use Spoofed ELO",
    }
    -- Spanish labels map (display only)
    local RankLabels = {
        ["Archnemesis"] = "Archnemesis",
        ["Nemesis"] = "Nemesis",
        ["Onyx 3"] = "Onix 3", ["Onyx 2"] = "Onix 2", ["Onyx 1"] = "Onix 1",
        ["Diamond 3"] = "Diamante 3", ["Diamond 2"] = "Diamante 2", ["Diamond 1"] = "Diamante 1",
        ["Platinum 3"] = "Platino 3", ["Platinum 2"] = "Platino 2", ["Platinum 1"] = "Platino 1",
        ["Gold 3"] = "Oro 3", ["Gold 2"] = "Oro 2", ["Gold 1"] = "Oro 1",
        ["Silver 3"] = "Plata 3", ["Silver 2"] = "Plata 2", ["Silver 1"] = "Plata 1",
        ["Bronze 3"] = "Bronce 3", ["Bronze 2"] = "Bronce 2", ["Bronze 1"] = "Bronce 1",
        ["Unranked"] = "Unranked",
        ["Use Spoofed ELO"] = "Usar ELO spoofeado",
    }

    local function ApplyRankSpoof(rankName)
        rankName = rankName or "Archnemesis"
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        local eloMap = nil
        pcall(function()
            -- kCharmEloMap is inside source scope; try getgenv
            eloMap = getgenv()._ShakoCharmEloMap
        end)
        -- fallback elo values (same as AURORA)
        local fallback = {
            ["Unranked"] = -1, ["Bronze 1"] = 0, ["Bronze 2"] = 200, ["Bronze 3"] = 400,
            ["Silver 1"] = 600, ["Silver 2"] = 800, ["Silver 3"] = 1000,
            ["Gold 1"] = 1200, ["Gold 2"] = 1400, ["Gold 3"] = 1600,
            ["Platinum 1"] = 1800, ["Platinum 2"] = 2000, ["Platinum 3"] = 2200,
            ["Diamond 1"] = 2400, ["Diamond 2"] = 2600, ["Diamond 3"] = 2800,
            ["Onyx 1"] = 3000, ["Onyx 2"] = 3200, ["Onyx 3"] = 3400,
            ["Nemesis"] = 3600, ["Archnemesis"] = 3600,
        }
        local elo = (eloMap and eloMap[rankName]) or fallback[rankName] or 3600

        if RivalsSpoof then
            RivalsSpoof.charm = RivalsSpoof.charm or {}
            RivalsSpoof.charm.charm_rank = rankName
            -- also push ELO on leaderboard so rank shows
            if RivalsSpoof.lb then
                RivalsSpoof.lb.elo = true
                RivalsSpoof.lb.elo_value = elo
            end
        end
        if st then
            st.charm = st.charm or {}
            st.charm.charm_rank = rankName
            -- enable all seasons so rank shows everywhere
            st.charm.s0_charm = true
            st.charm.s1_charm = true
            st.charm.s2_charm = true
            st.charm.s3_charm = true
            if st.leaderboard then
                st.leaderboard.ELO = true
                st.leaderboard.elo_value = elo
            end
        end
        -- seasons ON (como source: hay que tenerlas activas)
        if RivalsSpoof and RivalsSpoof.charm then
            RivalsSpoof.charm.s0_charm = true
            RivalsSpoof.charm.s1_charm = true
            RivalsSpoof.charm.s2_charm = true
            RivalsSpoof.charm.s3_charm = true
            RivalsSpoof.charm.arch_rank = RivalsSpoof.charm.arch_rank or 1
        end
        pcall(function()
            if getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state then
                local sc = getgenv()._ShakoStates.spoofer_state.charm
                if sc then
                    sc.s0_charm, sc.s1_charm, sc.s2_charm, sc.s3_charm = true, true, true, true
                end
            end
        end)
        pcall(RivalsApplyLeaderboard)
        pcall(function()
            if HookSeasonRankCharm then HookSeasonRankCharm() end
            if getgenv()._ShakoHookCharm then getgenv()._ShakoHookCharm() end
        end)
        pcall(RivalsUpdateCharm)
        -- AURORA-style: forzar ELO en leaderboard serial + atributo
        pcall(function()
            if getgenv()._ShakoUpdateLB then getgenv()._ShakoUpdateLB() end
        end)
        pcall(function()
            LocalPlayer:SetAttribute("StatisticHighestELO", elo)
            LocalPlayer:SetAttribute("HighestELO", elo)
        end)
        -- re-aplicar charm varias veces (UI carga tarde)
        task.spawn(function()
            for _ = 1, 5 do
                task.wait(0.4)
                pcall(function()
                    if getgenv()._ShakoHookCharm then getgenv()._ShakoHookCharm() end
                    if getgenv()._ShakoUpdateCharm then getgenv()._ShakoUpdateCharm() end
                end)
            end
        end)
        Notify("Rank", (RankLabels[rankName] or rankName) .. " · ELO " .. tostring(elo) .. " · seasons ON")
    end

    local RankBox = Tabs.Spoofer:AddRightGroupbox("Rank Spoofer")
    RankBox:AddDropdown("RankSelect", {
        Text = "Elegir Rango",
        Values = kCharmRanks,
        Default = 1, -- Archnemesis
        Callback = function(v)
            ApplyRankSpoof(v)
        end
    })
    RankBox:AddSlider("ArchRankNum", {
        Text = "Arch Rank # (solo Archnemesis)", Default = 1, Min = 1, Max = 999999999, Rounding = 0,
        Callback = function(v)
            if RivalsSpoof and RivalsSpoof.charm then RivalsSpoof.charm.arch_rank = v end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.charm then st.charm.arch_rank = v end
            RivalsUpdateCharm()
        end
    })
    RankBox:AddToggle("RankS0", { Text = "Mostrar Season 0", Default = true, Callback = function(v)
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        if st and st.charm then st.charm.s0_charm = v end
        if RivalsSpoof and RivalsSpoof.charm then RivalsSpoof.charm.s0_charm = v end
        RivalsUpdateCharm()
    end })
    RankBox:AddToggle("RankS1", { Text = "Mostrar Season 1", Default = true, Callback = function(v)
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        if st and st.charm then st.charm.s1_charm = v end
        if RivalsSpoof and RivalsSpoof.charm then RivalsSpoof.charm.s1_charm = v end
        RivalsUpdateCharm()
    end })
    RankBox:AddToggle("RankS2", { Text = "Mostrar Season 2", Default = true, Callback = function(v)
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        if st and st.charm then st.charm.s2_charm = v end
        if RivalsSpoof and RivalsSpoof.charm then RivalsSpoof.charm.s2_charm = v end
        RivalsUpdateCharm()
    end })
    RankBox:AddToggle("RankS3", { Text = "Mostrar Season 3", Default = true, Callback = function(v)
        local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
        if st and st.charm then st.charm.s3_charm = v end
        if RivalsSpoof and RivalsSpoof.charm then RivalsSpoof.charm.s3_charm = v end
        RivalsUpdateCharm()
    end })
    RankBox:AddButton({
        Text = "Aplicar Rango (Archnemesis etc)",
        Func = function()
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            local r = (st and st.charm and st.charm.charm_rank) or (RivalsSpoof and RivalsSpoof.charm and RivalsSpoof.charm.charm_rank) or "Archnemesis"
            ApplyRankSpoof(r)
        end
    })
    RankBox:AddButton({
        Text = "Forzar Archnemesis #1",
        Func = function()
            if RivalsSpoof and RivalsSpoof.charm then RivalsSpoof.charm.arch_rank = 1 end
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.charm then st.charm.arch_rank = 1 end
            ApplyRankSpoof("Archnemesis")
        end
    })
    RankBox:AddLabel("1) Elige rango  2) Activa seasons  3) Aplicar")
    RankBox:AddLabel("Método AURORA: FormatSeasonRankCharm + ELO 3600")

    -- keep legacy CharmBox name pointing same (compat)
    local CharmBox = RankBox

    local MPBox = Tabs.Misc:AddRightGroupbox("marketplace")
    MPBox:AddToggle("FakeRobux", {
        Text = "Fake Robux", Default = false,
        Callback = function(v)
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.marketplace then st.marketplace.fake_robux = v end
            RivalsUpdateMarketplace()
        end
    })
    MPBox:AddInput("RobuxAmount", {
        Text = "Robux Amount", Default = "999999", Numeric = true,
        Callback = function(v)
            local n = tonumber(v) or 0
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.marketplace then st.marketplace.robux_amount = n end
            RivalsUpdateMarketplace()
        end
    })
    MPBox:AddToggle("FakeGifting", {
        Text = "Fake Gifting", Default = false,
        Callback = function(v)
            local st = getgenv()._ShakoStates and getgenv()._ShakoStates.spoofer_state
            if st and st.marketplace then st.marketplace.fake_gifting = v end
            RivalsUpdateMarketplace()
        end
    })
    MPBox:AddButton({ Text = "Apply Marketplace", Func = function() RivalsUpdateMarketplace() end })
end)

-- ===== UNLOCK TAB (AURORA completo — siempre visible) =====
pcall(function()
    if not Tabs.Unlock then
        local ok, tab = pcall(function() return Window:AddTab("Unlock", "unlock") end)
        if ok and tab then Tabs.Unlock = tab end
    end
    if not Tabs.Unlock then
        -- fallback dentro de Spoofer
        Tabs.Unlock = Tabs.Spoofer
    end

    local function skinState()
        local st = getgenv()._ShakoStates
        return st and st.skinchanger_state
    end

    local Unlocker = Tabs.Unlock:AddLeftGroupbox("Unlocker (AURORA)")
    Unlocker:AddDropdown("UnlockType", {
        Text = "Type",
        Values = {"Skin", "Wrap", "Charm", "Finisher"},
        Default = 1,
        Callback = function(v)
            local s = skinState()
            if s then s.general_unlocker.unlock_type = v end
        end
    })
    Unlocker:AddDropdown("UnlockRarity", {
        Text = "Rarity",
        Values = {"Common", "Rare", "Legendary", "Mythical", "Unique", "Unobtainable"},
        Default = 1,
        Callback = function(v)
            local s = skinState()
            if s then s.general_unlocker.unlock_rarity = v end
        end
    })
    Unlocker:AddButton({
        Text = "Unlock Selected Rarity",
        Func = function()
            if getgenv()._ShakoUnlockRarity then
                local ok, err = pcall(getgenv()._ShakoUnlockRarity)
                Notify("Unlock", ok and "Rarity OK" or tostring(err))
            else
                Notify("Unlock", "fn no lista")
            end
        end
    })
    Unlocker:AddButton({
        Text = "Lock Selected Rarity",
        Func = function()
            if getgenv()._ShakoLockRarity then pcall(getgenv()._ShakoLockRarity) end
            Notify("Unlock", "Lock rarity")
        end
    })
    Unlocker:AddButton({
        Text = "Unlock All of Type",
        Func = function()
            if getgenv()._ShakoUnlockAllOfType then
                local ok, err = pcall(getgenv()._ShakoUnlockAllOfType)
                Notify("Unlock", ok and "Type OK" or tostring(err))
            else
                Notify("Unlock", "UnlockAllOfType no lista")
            end
        end
    })
    Unlocker:AddButton({
        Text = "Lock All of Type",
        Func = function()
            if getgenv()._ShakoLockAllOfType then pcall(getgenv()._ShakoLockAllOfType) end
            Notify("Unlock", "Lock type")
        end
    })

    local Specific = Tabs.Unlock:AddRightGroupbox("Specific")
    Specific:AddDropdown("SpecificType", {
        Text = "Type",
        Values = {"Skin", "Wrap", "Charm", "Finisher"},
        Default = 1,
        Callback = function(v)
            local s = skinState()
            if s then s.specific_unlocker.unlock_type = v end
        end
    })
    Specific:AddInput("SpecificCosmetic", {
        Text = "Cosmetic name",
        Default = "",
        Placeholder = "e.g. 10B Visits",
        Callback = function(v)
            local s = skinState()
            if s then s.specific_unlocker.cosmetic_name = v end
        end
    })
    Specific:AddInput("SpecificWeapon", {
        Text = "Weapon name",
        Default = "",
        Placeholder = "e.g. Assault Rifle",
        Callback = function(v)
            local s = skinState()
            if s then s.specific_unlocker.weapon_name = v end
        end
    })
    Specific:AddButton({
        Text = "Unlock Specific",
        Func = function()
            if getgenv()._ShakoUnlockSpecific then pcall(getgenv()._ShakoUnlockSpecific)
            else Notify("Unlock", "no specific fn") end
        end
    })
    Specific:AddButton({
        Text = "Lock Specific",
        Func = function()
            if getgenv()._ShakoLockSpecific then pcall(getgenv()._ShakoLockSpecific) end
        end
    })
    Specific:AddButton({
        Text = "Unlock Specific Weapon",
        Func = function()
            if getgenv()._ShakoUnlockSpecificWeapon then pcall(getgenv()._ShakoUnlockSpecificWeapon) end
        end
    })
    Specific:AddButton({
        Text = "Lock Specific Weapon",
        Func = function()
            if getgenv()._ShakoLockSpecificWeapon then pcall(getgenv()._ShakoLockSpecificWeapon) end
        end
    })

    local Equip = Tabs.Unlock:AddLeftGroupbox("Equip")
    Equip:AddDropdown("EquipType", {
        Text = "Type",
        Values = {"Skin", "Wrap", "Charm", "Finisher"},
        Default = 1,
        Callback = function(v)
            local s = skinState()
            if s then s.equip_unlocker.unlock_type = v end
        end
    })
    Equip:AddInput("EquipCosmetic", {
        Text = "Cosmetic", Default = "",
        Callback = function(v)
            local s = skinState()
            if s then s.equip_unlocker.cosmetic_name = v end
        end
    })
    Equip:AddInput("EquipWeapon", {
        Text = "Weapon", Default = "",
        Callback = function(v)
            local s = skinState()
            if s then s.equip_unlocker.weapon_name = v end
        end
    })
    Equip:AddToggle("EquipInverted", {
        Text = "Inverted", Default = false,
        Callback = function(v)
            local s = skinState()
            if s then s.equip_unlocker.inverted = v end
        end
    })
    Equip:AddButton({
        Text = "Apply Equip",
        Func = function()
            if getgenv()._ShakoEquipApply then pcall(getgenv()._ShakoEquipApply)
            else Notify("Equip", "no fn") end
        end
    })
    Equip:AddButton({
        Text = "Apply to all weapons",
        Func = function()
            if getgenv()._ShakoEquipApplyAll then pcall(getgenv()._ShakoEquipApplyAll) end
        end
    })

    local Bulk = Tabs.Unlock:AddRightGroupbox("Bulk")
    Bulk:AddButton({
        Text = "Unlock All Weapons",
        Func = function()
            if getgenv()._ShakoUnlockAllWeapons then pcall(getgenv()._ShakoUnlockAllWeapons)
            else Notify("Unlock", "no fn") end
        end
    })
    Bulk:AddButton({
        Text = "Lock All Weapons",
        Func = function()
            if getgenv()._ShakoLockAllWeapons then pcall(getgenv()._ShakoLockAllWeapons) end
        end
    })
    Bulk:AddButton({
        Text = "Unlock All for Weapon",
        Func = function()
            if getgenv()._ShakoUnlockAllForWeapon then pcall(getgenv()._ShakoUnlockAllForWeapon) end
        end
    })
    Bulk:AddButton({
        Text = "Lock All for Weapon",
        Func = function()
            if getgenv()._ShakoLockAllForWeapon then pcall(getgenv()._ShakoLockAllForWeapon) end
        end
    })
    Bulk:AddButton({
        Text = "UNLOCK ALL",
        Func = function() RivalsUnlockAll() end
    })
    Bulk:AddButton({
        Text = "LOCK ALL",
        Func = function() RivalsLockAll() end
    })
    Bulk:AddLabel("Igual que AURORA UnlockerPage")
end)

-- Misc + Configs (siempre se construyen aunque Spoofer falle)
pcall(function()
if not Tabs.Misc then return end
local ACBox = Tabs.Misc:AddLeftGroupbox("Anti-Kick / Misc")
ACBox:AddToggle("AntiKick", {
    Text = "Anti Kick (cliente)", Default = true,
    Callback = function(v) config.AntiKick = v end
})
ACBox:AddToggle("BlockIllegalNotify", {
    Text = "Notify kicks", Default = true,
    Callback = function(v) config.BlockIllegalNotify = v end
})
ACBox:AddToggle("AutoRespawn", {
    Text = "Auto Respawn", Default = false,
    Callback = function(v) config.AutoRespawn = v end
})
ACBox:AddButton({
    Text = "Rejoin",
    Func = function() pcall(function() TeleportService:Teleport(game.PlaceId, LocalPlayer) end) end
})

-- HIT / KILL SONGS UI
local SoundBox = Tabs.Misc:AddLeftGroupbox("Hit / Kill Sounds")
SoundBox:AddToggle("HitSoundEnabled", {
    Text = "Hit Sounds", Default = true,
    Callback = function(v) config.HitSoundEnabled = v end
})
SoundBox:AddToggle("KillSoundEnabled", {
    Text = "Kill Sounds", Default = true,
    Callback = function(v) config.KillSoundEnabled = v end
})
SoundBox:AddDropdown("HitSoundMode", {
    Text = "Hit Mode",
    Values = {"Random", "Selected"},
    Default = 1,
    Callback = function(v) config.HitSoundMode = v end
})
SoundBox:AddDropdown("SelectedHitSound", {
    Text = "Hit Sound (Grief)",
    Values = {"Rust HS","Neverlose","Gamesense","Skeet","Fatality","Bonk","Osu","CSGO","TF2 Critical","TF2","Minecraft","Minecraft Hit","Bubble","Bruh","Vine","Among Us","Click","Ding","Beep","Laser","UwU","Weeb","Bamboo","Crowbar","Bat","Saber","Stone","Snow","Steve","Mario","Call of Duty","Baimware","RIFK7","Old Fatality","Big","Fart","Sans","Pick","Pop","Rust","One","Soft Bell","Minecraft Bow","Bambi","Slime"},
    Default = 1,
    Callback = function(v) config.SelectedHitSound = v end
})

SoundBox:AddDropdown("KillSoundMode", {
    Text = "Kill Mode",
    Values = {"Random", "Selected"},
    Default = 1,
    Callback = function(v) config.KillSoundMode = v end
})
SoundBox:AddDropdown("SelectedKillSound", {
    Text = "Kill Sound",
    Values = {"OOF","New Death","Wake Up Bozo","Emotional Dmg","You Are Bad","Alone","Fatality","Vine","Among Us","TF2 Critical","Neverlose","Rust HS"},
    Default = 1,
    Callback = function(v) config.SelectedKillSound = v end
})

SoundBox:AddSlider("HitSoundVolume", {
    Text = "Hit Volume", Default = 2, Min = 0.1, Max = 10, Rounding = 1,
    Callback = function(v) config.HitSoundVolume = v end
})
SoundBox:AddSlider("HitSoundPitch", {
    Text = "Hit Pitch (AURORA)", Default = 1, Min = 0.1, Max = 10, Rounding = 1,
    Callback = function(v) config.HitSoundPitch = v end
})
SoundBox:AddSlider("KillSoundVolume", {
    Text = "Kill Volume", Default = 3, Min = 0.1, Max = 10, Rounding = 1,
    Callback = function(v) config.KillSoundVolume = v end
})
SoundBox:AddButton({
    Text = "Test Hit Sound",
    Func = function()
        -- fuerza play aunque el toggle esté off (solo test)
        local id = (SFX.hitSounds and SFX.hitSounds[config.SelectedHitSound or "Rust HS"]) or 12222216
        SFX.play(id, math.max(config.HitSoundVolume or 7, 5), config.HitSoundPitch or 1)
        Notify("Hit", "Test id=" .. tostring(id))
    end
})
SoundBox:AddButton({
    Text = "Test TF2 Crit",
    Func = function()
        SFX.play(296102734, math.max(config.HitSoundVolume or 7, 5), 1)
        Notify("Hit", "TF2 Crit")
    end
})
SoundBox:AddButton({
    Text = "Test Kill OOF",
    Func = function()
        SFX.play(15666462, math.max(config.KillSoundVolume or 8, 5), 1)
        Notify("Kill", "OOF")
    end
})
SoundBox:AddButton({
    Text = "Test Click (garantizado)",
    Func = function()
        SFX.play(12222216, 10, 1)
        Notify("Hit", "Click 12222216 volume 10")
    end
})
SoundBox:AddButton({
    Text = "Test All Hits",
    Func = function()
        task.spawn(function()
            for _, n in ipairs(SFX.hitNames) do
                SFX.play(SFX.hitSounds[n], 2)
                Notify("Hit", n)
                task.wait(0.7)
            end
        end)
    end
})
SoundBox:AddLabel("Si no suena: sube Volume o prueba TF2 Crit / OOF")

local KBBox = Tabs.Misc:AddRightGroupbox("Keybinds (DESACTIVADAS)")
KBBox:AddLabel("Sin teclas — solo UI")
KBBox:AddLabel("Slide (C) ya no activa Strafe")
if false then -- keybind UI oculto
function kbDrop(id, text, cfgKey, default)
    KBBox:AddDropdown(id, {
        Text = text,
        Values = {"Q","E","R","T","Y","U","F","G","H","B","C","X","Z","V","LeftAlt","LeftControl","CapsLock"},
        Default = default,
        Callback = function(v) config[cfgKey] = v end
    })
end
kbDrop("KBStick", "Stick / shako", "KeybindStick", 14)
kbDrop("KBRage", "Rage", "KeybindRage", 3)
kbDrop("KBSilent", "Silent Aim", "KeybindSilent", 4)
kbDrop("KBAimbot", "Aimbot", "KeybindAimbot", 5)
kbDrop("KBFly", "Fly", "KeybindFly", 7)
kbDrop("KBNoclip", "Noclip", "KeybindNoclip", 8)
kbDrop("KBAntiAim", "Anti Aim (spin)", "KeybindAntiAim", 10)
kbDrop("KBStrafe", "Strafe", "KeybindStrafe", 11)
kbDrop("KBOrbit", "Orbit", "KeybindOrbit", 12)
kbDrop("KBSpeed", "Speed", "KeybindSpeed", 13)
kbDrop("KBVoid", "Void Spam", "KeybindVoidSpam", 9)
KBBox:AddLabel("KEYBINDS OFF — solo UI (slide no activa strafe)")
end -- keybind UI

-- Theme: default Obsidian (sin override)
end) -- end Misc pcall

pcall(function()
if not Tabs.Configs then
    pcall(function()
        if Window and Window.AddTab then
            Tabs.Configs = Window:AddTab("Configs")
        end
    end)
end
if not Tabs.Configs then return end

local CfgBox = Tabs.Settings:AddLeftGroupbox("configs")
local configNameInput = "default"
local selectedConfig = "(ninguna)"

CfgBox:AddInput("ConfigName", {
    Text = "Nombre", Default = "default",
    Callback = function(v) configNameInput = tostring(v or "default") end
})
local configDropdown = CfgBox:AddDropdown("ConfigSelect", {
    Text = "Elegir", Values = (Cfg and Cfg.List and Cfg.List()) or {"(ninguna)"}, Default = 1,
    Callback = function(v) selectedConfig = v end
})
local function RefreshCfg()
    pcall(function()
        local list = Cfg.List()
        if configDropdown.SetValues then configDropdown:SetValues(list) end
    end)
end
CfgBox:AddButton({ Text = "Refrescar lista", Func = RefreshCfg })
CfgBox:AddButton({
    Text = "Guardar (nuevo)",
    Func = function()
        local n = configNameInput ~= "" and configNameInput or "default"
        if Cfg.Save(n, false) then
            Notify("Config", "Guardada: " .. n)
            RefreshCfg()
        else
            Notify("Config", "Error al guardar (¿ya existe? usa Overwrite)")
        end
    end
})
CfgBox:AddButton({
    Text = "Overwrite",
    Func = function()
        local n = (selectedConfig and selectedConfig ~= "(ninguna)") and selectedConfig or configNameInput
        if Cfg.Save(n, true) then
            Notify("Config", "Overwrite: " .. tostring(n))
            RefreshCfg()
        end
    end
})
CfgBox:AddButton({
    Text = "Load",
    Func = function()
        local n = (selectedConfig and selectedConfig ~= "(ninguna)") and selectedConfig or configNameInput
        if Cfg.Load(n) then
            Notify("Config", "Cargada: " .. tostring(n))
        else
            Notify("Config", "No se pudo cargar")
        end
    end
})
CfgBox:AddButton({
    Text = "Delete",
    Func = function()
        if selectedConfig and selectedConfig ~= "(ninguna)" then
            if Cfg.Delete(selectedConfig) then
                selectedConfig = "(ninguna)"
                Notify("Config", "Borrada")
                RefreshCfg()
            end
        end
    end
})
CfgBox:AddButton({
    Text = "Set Auto Load",
    Func = function()
        local n = (selectedConfig and selectedConfig ~= "(ninguna)") and selectedConfig or configNameInput
        Cfg.SetAutoLoad(n)
        Notify("Config", "AutoLoad → " .. tostring(n))
    end
})
CfgBox:AddButton({
    Text = "Remove Auto Load",
    Func = function()
        Cfg.RemoveAutoLoad()
        Notify("Config", "AutoLoad quitado")
    end
})
CfgBox:AddButton({
    Text = "Ver Auto Load",
    Func = function()
        local n = Cfg.GetAutoLoad()
        Notify("Config", n and ("Auto: " .. n) or "Sin auto load")
    end
})

-- SaveManager / Theme de UEObsidian (si cargó)
pcall(function()
    if SaveManager and SaveManager.BuildConfigSection then
        local smBox = Tabs.Configs:AddRightGroupbox("SaveManager (Obsidian)")
        SaveManager:SetLibrary(Library)
        SaveManager:SetFolder("shako_win")
        pcall(function() SaveManager:IgnoreThemeSettings() end)
        SaveManager:BuildConfigSection(smBox)
    end
end)
pcall(function()
    if ThemeManager and ThemeManager.ApplyToGroupbox then
        local th = Tabs.Configs:AddRightGroupbox("Theme")
        ThemeManager:ApplyToGroupbox(th)
    elseif ThemeManager and ThemeManager.ApplyToTab then
        ThemeManager:ApplyToTab(Tabs.Configs)
    end
end)

CfgBox:AddLabel("Guarda toggles + spoof IDs + avatar")






RefreshCfg()
end) -- Configs pcall
end -- BuildUI
pcall(function() BuildUI() end)
if not Window then warn("[shako] UI fail") return end

-- Keybinds eliminadas (slide C ya no activa Strafe)
-- Todo se controla solo desde la UI Obsidian

LocalPlayer.CharacterAdded:Connect(function(char)
    if config.ForceFieldVisual then
        task.defer(function()
            task.wait(0.4)
            ApplyForceField(char)
        end)
    end
    CanMove = false
    SpawnTime = tick()
    AntiUnnamedReady = false
    AntiUnnamedStartTime = tick()
    LastAUCharacter = char
    HookCharacter(char)
    task.wait(config.SpawnDelay or 4.7)
    CanMove = true
    ResetCameraNormal()
    if config.AntiUnnamed then
        ArmAntiUnnamedOnSpawn()
        StartAUWatcher()
    else
        if config.Enabled or config.StrafeEnabled or config.OrbitEnabled then StartMainLoop() end
        if config.RagebotEnabled then StartRagebot() end
        if config.VoidSpamEnabled then StartVoidSpam() end
    end
    if config.FlyEnabled then StartFly() end
    if config.NoclipEnabled then StartNoclip() end
    if config.AntiAimEnabled then StartAntiAim() end
    if config.AimbotEnabled then StartAimbot() end
    if config.AutoClickEnabled then StartAutoClick() end
    if config.NoRecoil then StartNoRecoil() end
    if config.RapidFire then StartRapidFire() end
    if config.FastMelee then StartFastMelee() end
    if config.PredDodgeEnabled then StartPredDodge() end
    if config.TriggerbotEnabled then StartTriggerbot() end
    if config.ESPEnabled then task.defer(function() pcall(StartESP) end) end
    lastHurtHP = 100
    if anyDefenseOn() then StartDefense() end
end)

if LocalPlayer.Character then
    HookCharacter(LocalPlayer.Character)
    LastAUCharacter = LocalPlayer.Character
end

SetupAntiKick()
task.defer(ResetCameraNormal)

CanMove = false
SpawnTime = tick()
task.spawn(function()
    task.wait(math.min(config.SpawnDelay or 2, 3))
    CanMove = true
    pcall(function()
        if config.Enabled or config.StrafeEnabled or config.OrbitEnabled then
            StartMainLoop()
        end
        if config.UECounterEnabled or config.AntiUnnamed then
            StartUECounter()
            StartAUWatcher()
        end
        if config.ESPEnabled then
            task.wait(0.5)
            pcall(StartESP)
        end
        if config.NoRecoil then StartNoRecoil() end
        if config.RapidFire then StartRapidFire() end
        if config.FastMelee then StartFastMelee() end
        if config.SlideBoostEnabled then StartSlideBoostLoop() end
        config.StretchedRes = false -- removed
        ResetCameraNormal()
        EnsureCameraFree()
    end)
end)

Notify("shako.win", "v5.32.14c · hard FPS")
print("shako.win v5.32.14c hard FPS")
pcall(function() task.defer(function()
    getgenv()._ShakoQuiet = true
    pcall(Cfg.TryAutoLoad)
    task.wait(2)
    getgenv()._ShakoQuiet = false
end) end)


pcall(function() getgenv().shako_win_ok = true end)
