local Players         = game:GetService("Players")
local Workspace       = game:GetService("Workspace")
local HttpService     = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local RS              = game:GetService("ReplicatedStorage")
local vim             = game:GetService("VirtualInputManager")

vim:SendKeyEvent(true, Enum.KeyCode.F9, false, game)
task.wait()
vim:SendKeyEvent(false, Enum.KeyCode.F9, false, game)


local WEBHOOK_URL   = ""
local WEBHOOK_500K  = ""
local WEBHOOK_MID   = ""
local WEBHOOK_HIGH  = ""
local WEBHOOK_ULTRA = ""
local CARPET_URL    = ""
local FIREBASE_URL  = ""

local TARGET_NAMES    = {""}
local ULTRA_BRAINROTS = {"Dragon Cannelloni", "Cerberus", "La Secret Combinasion", "Strawberry Elephant", "Meowl", "Skibidi Toilet"}
local BLACK_NAMES     = {"", ""}

local MIN_VALUE     = 500000
local CARPET_MINGEN = 10000
local NOTIFIER_NAME = "Cannelloni Notify"
local CURRENT_JOB   = tostring(game.JobId)
local PLACE_ID      = game.PlaceId
local EMBED_COLOR   = 16776960

local LocalPlayer = Players.LocalPlayer

print("Players: " .. tostring(#Players:GetPlayers()))

local http_request = http_request or (syn and syn.request) or (http and http.request) or request

local XOR_KEY = {
    0x7C, 0x92, 0x24, 0x19, 0xF6, 0x47, 0x81, 0x31,
    0x2F, 0x08, 0xDE, 0x00, 0xA7, 0x10, 0x6D, 0x7F,
    0x4A, 0x8B, 0x3C, 0x9F, 0xE2, 0x55, 0x1D, 0x6A,
    0x9C, 0x33, 0x7E, 0xB4, 0xD8, 0x2F, 0x91, 0x46
}

local function bxor(a, b)
    local r, m = 0, 1
    while a > 0 or b > 0 do
        if (a % 2) ~= (b % 2) then r = r + m end
        a = math.floor(a / 2)
        b = math.floor(b / 2)
        m = m * 2
    end
    return r
end

local function encryptXOR(text)
    local bytes = {}
    for i = 1, #text do
        local byte    = string.byte(text, i)
        local keyByte = XOR_KEY[((i - 1) % #XOR_KEY) + 1]
        bytes[i]      = bxor(byte, keyByte)
    end
    local hex = ""
    for _, b in ipairs(bytes) do
        hex = hex .. string.format("%02X", b)
    end
    return hex
end

local function getEncryptedJobId(jobId)
    local data = HttpService:JSONEncode({
        jobId   = jobId,
        placeId = PLACE_ID,
        sender  = LocalPlayer and LocalPlayer.Name or "Unknown",
    })
    return encryptXOR(data)
end

-- ================================================================
-- LOAD MODULES
-- ================================================================
local Synchronizer  = nil
local AnimalsData   = nil
local AnimalsShared = nil

do
    local function tryLoad(parent, name)
        local ok, result = pcall(function()
            return require(parent:WaitForChild(name, 5))
        end)
        return ok and result or nil
    end
    local ok_pkg, Packages = pcall(function() return RS:WaitForChild("Packages", 5) end)
    local ok_dat, Datas    = pcall(function() return RS:WaitForChild("Datas",    5) end)
    local ok_sh,  Shared   = pcall(function() return RS:WaitForChild("Shared",   5) end)
    if ok_pkg and Packages then Synchronizer  = tryLoad(Packages, "Synchronizer") end
    if ok_dat and Datas    then AnimalsData   = tryLoad(Datas,    "Animals")      end
    if ok_sh  and Shared   then AnimalsShared = tryLoad(Shared,   "Animals")      end
end

-- ================================================================
-- FUNÇÕES AUXILIARES
-- ================================================================
local function parseValue(text)
    if not text then return 0 end
    local clean = tostring(text):gsub("[%$%s%,%/s%/m%/h]", "")
    local numStr, suf = clean:match("([%d%.]+)(%a?)")
    if not numStr then return 0 end
    local n    = tonumber(numStr) or 0
    local mult = ({K=1e3, M=1e6, B=1e9, T=1e12})[(suf or ""):upper()] or 1
    return n * mult
end

local function formatValue(n)
    if n >= 1e12 then return string.format("%.1fT", n/1e12) end
    if n >= 1e9  then return string.format("%.1fB", n/1e9)  end
    if n >= 1e6  then return string.format("%.1fM", n/1e6)  end
    if n >= 1e3  then return string.format("%.1fK", n/1e3)  end
    return tostring(math.floor(n))
end

local function isBlacklisted(itemName)
    local low = itemName:lower()
    for _, b in ipairs(BLACK_NAMES) do
        if b ~= "" and low:find(b:lower()) then return true end
    end
    return false
end

local function isTargeted(itemName)
    local low = itemName:lower()
    for _, t in ipairs(TARGET_NAMES) do
        if t ~= "" and low:find(t:lower()) then return true end
    end
    return false
end

local function shouldScan(name, genValue)
    if isBlacklisted(name) then return false end
    return genValue >= MIN_VALUE or isTargeted(name)
end

local function shouldScanCarpet(name, genValue)
    if isBlacklisted(name) then return false end
    return genValue >= CARPET_MINGEN or isTargeted(name)
end

local function resolveOwnerName(owner)
    if not owner then return nil end
    if typeof(owner) == "Instance" and owner:IsA("Player") then
        return owner.Name
    elseif typeof(owner) == "table" then
        if owner.Name and owner.Name ~= "" then return owner.Name end
        if owner.UserId then
            for _, p in ipairs(Players:GetPlayers()) do
                if p.UserId == owner.UserId then return p.Name end
            end
            return tostring(owner.UserId)
        end
    elseif typeof(owner) == "Instance" then
        return owner.Name
    end
    return nil
end

-- ================================================================
-- IGNORE FUSE
-- ================================================================
local function isFusing(animalData)
    if animalData.Machine then
        if animalData.Machine.Type == "Fuse" and animalData.Machine.Active then
            return true
        end
    end
    return false
end

-- ================================================================
-- DUEL DETECTION
-- ================================================================
local function isInDuel(animalData)
    local data = animalData.Data or animalData
    if animalData.Machine and type(animalData.Machine) == "table" then
        local mType = animalData.Machine.Type
        if type(mType) == "string" and mType:lower():find("duel") then return true end
    end
    if data and type(data) == "table" and data.Machine and type(data.Machine) == "table" then
        local mType = data.Machine.Type
        if type(mType) == "string" and mType:lower():find("duel") then return true end
    end
    if animalData.InDuel == true or animalData.inDuel == true or animalData.in_duel == true then return true end
    if type(data) == "table" then
        if data.InDuel == true or data.inDuel == true then return true end
    end
    return false
end

local function cameBackFromDuel(player)
    if player == LocalPlayer then
        local ok, data = pcall(function()
            return TeleportService:GetLocalPlayerTeleportData()
        end)
        if ok and type(data) == "table" then
            if data.fromDuel or data.DuelData or data.duel then return true end
        end
    end
    for _, child in ipairs(player:GetChildren()) do
        local nameLower = child.Name:lower()
        if nameLower:find("duel") or nameLower:find("returned") then return true end
    end
    local attrOk, attrs = pcall(function() return player:GetAttributes() end)
    if attrOk and attrs then
        for key, val in pairs(attrs) do
            if key:lower():find("duel") or key:lower():find("returned") then
                if val == true or val == 1 then return true end
            end
        end
    end
    return false
end

local function getDuelPlayers()
    local duelPlayers = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if cameBackFromDuel(player) then
            duelPlayers[player.Name]             = true
            duelPlayers[tostring(player.UserId)] = true
        end
    end
    return duelPlayers
end

local function getPlotOwner(plot)
    local ownerAttr = plot:GetAttribute("Owner") or plot:GetAttribute("PlayerName")
    if ownerAttr then
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Name == tostring(ownerAttr) or tostring(p.UserId) == tostring(ownerAttr) then
                return p
            end
        end
    end
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Name == plot.Name then return p end
    end
    return nil
end

local function isPlotOwnerInDuel(plot, duelPlayers)
    local owner = plot:GetAttribute("Owner") or plot:GetAttribute("PlayerName") or plot.Name
    if duelPlayers[owner] or duelPlayers[tostring(owner)] then return true end
    local player = getPlotOwner(plot)
    if player and (duelPlayers[player.Name] or duelPlayers[tostring(player.UserId)]) then
        return true
    end
    return false
end

-- ================================================================
-- LOG DE DUPLICATAS
-- ================================================================
local loggedKeys = {}

local function hasBeenLogged(jobId, name, genText)
    local key = jobId .. "|" .. name .. "|" .. genText
    if loggedKeys[key] then return true end
    loggedKeys[key] = true
    return false
end

-- ================================================================
-- SCAN DE PLOTS
-- ================================================================
local function scanPlots(duelPlayers, seen, currentJobId)
    local results = {}

    local plots = Workspace:FindFirstChild("Plots")
    if not plots then return results end

    for _, plot in ipairs(plots:GetChildren()) do
        local ok, pot = pcall(function() return Synchronizer:Get(plot.Name) end)
        if not ok or not pot then continue end

        local ok2, list = pcall(function() return pot:Get("AnimalList") end)
        if not ok2 or type(list) ~= "table" then continue end

        local syncOwner = nil
        local ownerOk, ownerVal = pcall(function() return pot:Get("Owner") end)
        if ownerOk and ownerVal ~= nil then
            syncOwner = type(ownerVal) == "string" and ownerVal or tostring(ownerVal)
        end

        local plotDuel = false
        if syncOwner and duelPlayers[syncOwner] then
            plotDuel = true
        else
            plotDuel = isPlotOwnerInDuel(plot, duelPlayers)
        end

        local ownerName = syncOwner or resolveOwnerName(ownerVal) or "?"

        local plotItems         = {}
        local plotHasQualifying = false

        for _, animalData in pairs(list) do
            if type(animalData) ~= "table" then continue end
            if isFusing(animalData) then continue end

            local name = animalData.Index
            if not name then continue end
            if not AnimalsData or not AnimalsData[name] then continue end

            local inDuel = isInDuel(animalData) or plotDuel

            local data     = animalData.Data or animalData
            local mutation = data.Mutation
            if type(mutation) ~= "string" or mutation == "" then mutation = nil end

            local traitsTable = nil
            local traitsList  = {}
            if type(data.Traits) == "table" then
                traitsTable = {}
                if #data.Traits > 0 then
                    for _, trait in ipairs(data.Traits) do
                        if type(trait) == "string" then
                            table.insert(traitsTable, trait)
                            table.insert(traitsList, trait)
                        end
                    end
                else
                    for traitName, enabled in pairs(data.Traits) do
                        if enabled then
                            table.insert(traitsTable, traitName)
                            table.insert(traitsList, traitName)
                        end
                    end
                end
                if #traitsTable == 0 then traitsTable = nil end
            end

            local ok3, genValue = pcall(function()
                return AnimalsShared:GetGeneration(name, mutation, traitsTable, nil)
            end)
            if not ok3 or type(genValue) ~= "number" then continue end

            local qualifies = shouldScan(name, genValue)
            if qualifies then plotHasQualifying = true end

            local displayName = name
            local info = AnimalsData[name]
            if info and info.DisplayName then displayName = info.DisplayName end

            table.insert(plotItems, {
                name       = displayName,
                index      = name,
                genValue   = genValue,
                mutation   = mutation,
                traitsList = traitsList,
                traitCount = traitsTable and #traitsTable or 0,
                inDuel     = inDuel,
                qualifies  = qualifies,
                ownerName  = ownerName,
            })
        end

        if not plotHasQualifying then continue end

        for _, item in ipairs(plotItems) do
            if not item.qualifies then continue end

            local genText = "$" .. formatValue(item.genValue) .. "/s"
            local key     = item.name .. genText
            if seen[key] then continue end
            seen[key] = true

            if hasBeenLogged(currentJobId, item.name, genText) then continue end

            local isUltraName = false
            for _, u in ipairs(ULTRA_BRAINROTS) do
                if u ~= "" and item.name:lower() == u:lower() then
                    isUltraName = true
                    break
                end
            end

            local tier = (item.genValue >= 1e9 or isUltraName) and "ULTRA"
                      or (item.genValue >= 100e6)               and "HIGH"
                      or (item.genValue >= 10e6)                and "MID"
                      or (item.genValue >= 500e3)               and "500K"
                      or "LOW"

            table.insert(results, {
                tier      = tier,
                name      = item.name,
                gen       = genText,
                value     = item.genValue,
                mutation  = item.mutation,
                traits    = #item.traitsList > 0 and item.traitsList or nil,
                inDuel    = item.inDuel,
                ownerName = item.ownerName,
            })

            print(string.format(
                "FOUND: %s | Mutation: %s | Traits: %d | Gen: %s | Tier: %s%s",
                item.name,
                item.mutation or "None",
                item.traitCount,
                genText,
                tier,
                item.inDuel and " [IN DUEL]" or ""
            ))
        end
    end

    return results
end

-- ================================================================
-- SCAN DE CARPET
-- ================================================================
local function scanCarpet(seen, currentJobId)
    local results = {}

    for _, instance in ipairs(Workspace:GetChildren()) do
        if instance.ClassName ~= "Model" then continue end

        local name = instance:GetAttribute("Index")
        if not name then continue end
        if not AnimalsData or not AnimalsData[name] then continue end

        if isBlacklisted(name) then continue end

        local isFusingAttr = instance:GetAttribute("Fusing")
        if isFusingAttr == true then continue end

        local mutation = instance:GetAttribute("Mutation")
        if type(mutation) ~= "string" or mutation == "" then mutation = nil end

        local traitsTable = nil
        local traitsList  = {}
        local traitsRaw   = instance:GetAttribute("Traits")
        if traitsRaw and type(traitsRaw) == "string" then
            local ok, decoded = pcall(HttpService.JSONDecode, HttpService, traitsRaw)
            if ok and type(decoded) == "table" then
                traitsTable = {}
                if #decoded > 0 then
                    for _, trait in ipairs(decoded) do
                        if type(trait) == "string" then
                            table.insert(traitsTable, trait)
                            table.insert(traitsList, trait)
                        end
                    end
                else
                    for traitName, enabled in pairs(decoded) do
                        if enabled then
                            table.insert(traitsTable, traitName)
                            table.insert(traitsList, traitName)
                        end
                    end
                end
                if #traitsTable == 0 then traitsTable = nil end
            end
        end

        local ok3, genValue = pcall(function()
            return AnimalsShared:GetGeneration(name, mutation, traitsTable, nil)
        end)
        if not ok3 or type(genValue) ~= "number" then continue end

        if not shouldScanCarpet(name, genValue) then continue end

        local displayName = name
        local info = AnimalsData[name]
        if info and info.DisplayName then displayName = info.DisplayName end

        local genText    = "$" .. formatValue(genValue) .. "/s"
        local key        = "carpet|" .. displayName .. genText
        if seen[key] then continue end
        seen[key] = true

        if hasBeenLogged(currentJobId, "carpet|" .. name, genText) then continue end

        local traitCount = traitsTable and #traitsTable or 0

        table.insert(results, {
            name       = displayName,
            gen        = genText,
            value      = genValue,
            mutation   = mutation,
            traits     = #traitsList > 0 and traitsList or nil,
            traitCount = traitCount,
        })

        print(string.format(
            "FOUND (CARPET): %s | Mutation: %s | Traits: %d | Gen: %s",
            displayName,
            mutation or "None",
            traitCount,
            genText
        ))
    end

    return results
end

-- ================================================================
-- FIREBASE
-- ================================================================
local function sendToFirebase(itemName, itemValue, ownerName, jobId, inDuel)
    if FIREBASE_URL == "" then warn("❌ FIREBASE_URL vazia") return end

    local encryptedJob = getEncryptedJobId(jobId or CURRENT_JOB)

    local payload = {
        Date      = os.date("%d/%m/%Y %H:%M:%S"),
        IsInDuels = (inDuel == true),
        Item      = tostring(itemName or "Unknown"),
        JobId     = encryptedJob,
        Owner     = ownerName or "Unknown",
        PlaceId   = PLACE_ID,
        Players   = #Players:GetPlayers(),
        Timestamp = os.time(),
        Value     = tonumber(itemValue) or 0,
        ValueRaw  = formatValue(tonumber(itemValue) or 0),
    }

    local body = HttpService:JSONEncode(payload)
    print("📤 Enviando pro Firebase...")

    local success, response = pcall(function()
        return http_request({
            Url     = FIREBASE_URL,
            Method  = "PUT",
            Headers = { ["Content-Type"] = "application/json" },
            Body    = body,
        })
    end)

    if success and response then
        print("✅ Firebase OK | Status:", response.StatusCode)
    else
        warn("❌ Firebase falhou, tentando novamente...")
        task.wait(1)
        pcall(function()
            http_request({
                Url     = FIREBASE_URL,
                Method  = "PUT",
                Headers = { ["Content-Type"] = "application/json" },
                Body    = body,
            })
        end)
    end
end

-- ================================================================
-- WEBHOOK
-- ================================================================
local function getBrainrotImage(name)
    local ok, result = pcall(function()
        local url = "https://stealabrainrot.fandom.com/api.php?action=query&titles="
                    .. HttpService:UrlEncode(name)
                    .. "&prop=pageimages&piprop=original&format=json"
        local resp = http_request({ Url = url, Method = "GET" })
        if resp and resp.StatusCode == 200 then
            local data = HttpService:JSONDecode(resp.Body)
            for _, page in pairs(data.query.pages) do
                if page.original and page.original.source then return page.original.source end
            end
        end
    end)
    return ok and result or nil
end

-- Qualquer mutação vira emoji Discord: "Gold" → ":gold:", "Dark Matter" → ":dark_matter:"
local function getMutationPrefix(mutation)
    if not mutation then return "" end
    local m = mutation:lower()
    if m == "" then return "" end
    local emojiName = m:gsub("%s+", "_")
    return ":" .. emojiName .. ": "
end

local function buildListText(results, targetOwner)
    local listText = ""
    local count    = 0
    for _, r in ipairs(results) do
        if r.ownerName == targetOwner and count < 5 then
            local mutPrefix = getMutationPrefix(r.mutation)
            local line = "1x " .. mutPrefix .. r.name .. " (" .. r.gen .. ")"
            if r.inDuel then line = line .. " ⚔️" end
            listText = listText .. line .. "\n"
            count    = count + 1
        end
    end
    return listText
end

-- showOwner: se false, omite o field Owner no embed
local function sendWebhook(url, name, value, mutation, listText, showFields, footer, ownerName, jobId, inDuel, showOwner)
    if not url or url == "" then return end

    local imageUrl  = getBrainrotImage(name)
    local mutPrefix = getMutationPrefix(mutation)

    local description = "**Brainrots:**\n" .. (listText or "")

    if inDuel then
        description = description .. "\n⚔️ **Em Duel!**"
    end

    local embedTitle = "1x " .. mutPrefix .. name .. " (" .. formatValue(value) .. "/s)"

    local embed = {
        title       = embedTitle,
        description = description,
        color       = EMBED_COLOR,
        thumbnail   = imageUrl and { url = imageUrl } or nil,
        footer      = { text = footer or "Cannelloni Notify" },
        timestamp   = DateTime.now():ToIsoDate(),
    }

    if showFields then
        local fields = {
            { name = "Job ID",  value = "```" .. (jobId or CURRENT_JOB) .. "```", inline = false },
            { name = "Players", value = tostring(#Players:GetPlayers()),           inline = true  },
        }
        if showOwner ~= false and ownerName then
            table.insert(fields, { name = "Owner", value = tostring(ownerName), inline = true })
        end
        embed.fields = fields
    end

    pcall(function()
        http_request({
            Url     = url,
            Method  = "POST",
            Headers = { ["Content-Type"] = "application/json" },
            Body    = HttpService:JSONEncode({ username = NOTIFIER_NAME, embeds = { embed } }),
        })
    end)
end


-- ================================================================
-- EXECUÇÃO PRINCIPAL
-- ================================================================
task.wait(2)

local detectedJobId = game.JobId
local found         = false

if Synchronizer then
    local duelPlayers = getDuelPlayers()
    local seen        = {}

    -- SCAN DE PLOTS
    local results = scanPlots(duelPlayers, seen, detectedJobId)

    if #results > 0 then
        found = true
        table.sort(results, function(a, b) return a.value > b.value end)

        local best     = results[1]
        local listText = buildListText(results, best.ownerName)

        print("✅ Sucesso: " .. best.name .. " | Owner: " .. tostring(best.ownerName) .. " | " .. best.gen)
        if best.inDuel then print("⚔️ Em duel!") end

        -- Webhook principal: sempre envia, todos os tiers
        sendWebhook(WEBHOOK_URL, best.name, best.value, best.mutation, listText, true, "Cannelloni Notify",
                    best.ownerName, detectedJobId, best.inDuel, true)

        sendToFirebase(best.name, best.value, best.ownerName, detectedJobId, best.inDuel)

        local isUltraName = false
        for _, u in ipairs(ULTRA_BRAINROTS) do
            if u ~= "" and best.name:lower() == u:lower() then isUltraName = true break end
        end

        -- Webhooks por tier (além do principal)
        if best.value >= 1e9 or isUltraName then
            -- ULTRA: 1B+ ou nome ultra
            sendWebhook(WEBHOOK_ULTRA, best.name, best.value, best.mutation, listText, false, "Cannelloni AuraLights",
                        best.ownerName, detectedJobId, best.inDuel)

        elseif best.value >= 100e6 then
            -- HIGH: 100M – 999M
            sendWebhook(WEBHOOK_HIGH, best.name, best.value, best.mutation, listText, false, "Cannelloni HighLights",
                        best.ownerName, detectedJobId, best.inDuel)

        elseif best.value >= 10e6 then
            -- MID: 10M – 99M
            sendWebhook(WEBHOOK_MID, best.name, best.value, best.mutation, listText, false, "Cannelloni MidLights",
                        best.ownerName, detectedJobId, best.inDuel)

        elseif best.value >= 500e3 then
            sendWebhook(WEBHOOK_500K, best.name, best.value, best.mutation, listText, true, "Cannelloni Notify ・500k-10m",
                        best.ownerName, detectedJobId, best.inDuel, true)
        end
    end

    -- SCAN DE CARPET (ESTEIRA)
    if CARPET_URL and CARPET_URL ~= "" and AnimalsShared then
        local carpetResults = scanCarpet(seen, detectedJobId)

        if #carpetResults > 0 then
            found = true
            table.sort(carpetResults, function(a, b) return a.value > b.value end)

            local bestCarpet     = carpetResults[1]
            local carpetListText = ""
            for i, r in ipairs(carpetResults) do
                if i > 5 then break end
                local mutPrefix = getMutationPrefix(r.mutation)
                carpetListText  = carpetListText .. "1x " .. mutPrefix .. r.name .. " (" .. r.gen .. ")\n"
            end

            print("🟨 CARPET: " .. bestCarpet.name .. " | " .. bestCarpet.gen)

            sendWebhook(
                CARPET_URL,
                bestCarpet.name,
                bestCarpet.value,
                bestCarpet.mutation,
                carpetListText,
                true,                -- showFields: mostra Job ID e Players
                "Cannelloni Carpet",
                nil,                 -- sem owner
                detectedJobId,
                false,
                false                -- showOwner = false
            )
        end
    end
end

if not found then
    print("❌ Nenhum brainrot valioso encontrado")
    startInfiniteHop("Scan")
end

startInfiniteHop("Scan")
