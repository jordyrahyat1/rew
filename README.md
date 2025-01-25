local inv = {}
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local RP = game:GetService("ReplicatedStorage")
local TS = game:GetService("TweenService")
local Signal = require(RP.Storage.Modules.Signal)
local TroveModule = require(RP.Storage.Modules.Trove)
local Updater = require(script.Updater)
local Network = require(RP.Storage.Modules.Network)
local GridPack = require(script.Parent.Packages.GridPack)
local WeaponSizes = require(script.WeaponSizes)
local WeaponView = require(script.WeaponView)
local ViewportModel = require(script.Parent.ViewportModel)
local Rarity = require(RP.Storage.Modules.ToolRarity)
local ArmorDB = require(RP.Storage.Modules.Armor.Database)
local Sounds = RP.Storage.Sounds
local __class = {}
__class.__index = __class

export type SlotData = {Tool:Tool, Trove:TroveModule, Threads:{RBXScriptConnection}, Selected:boolean, Frame:Frame, Index:number}

local tfSlots = {
	[1] = Enum.KeyCode.One,
	[2] = Enum.KeyCode.Two,
	[3] = Enum.KeyCode.Three,
	[4] = Enum.KeyCode.Four,
	[5] = Enum.KeyCode.Five,
	[6] = Enum.KeyCode.Six,
	[7] = Enum.KeyCode.Seven,
	[8] = Enum.KeyCode.Eight,
	[9] = Enum.KeyCode.Nine,
}

function inv.New(player: Player)
	local self = setmetatable({}, __class)
	local pGui = player.PlayerGui.UI.HUD_GAMEPLAY.statsInfo
	local statsGui = player.PlayerGui.UI.HUD_GAMEPLAY.playerStats

	self.player = player
	self.toolEquiped = Signal.new()
	self.toolUnequiped = Signal.new()
	self.slots = {}
	self.lootgrid = nil
	self.enabled = true
	self.currentTool = nil
	self.ui = pGui
	self.statsui = statsGui
	self.currentAction = nil
	self.statsEnabled = false
	self.Backpack = nil
	self.storedItems = {}
	self.modifySlot = nil
	self.modifiedItem = nil
	self.alreadyLooted = {}
	self.lootItems = {}

	self.grid = GridPack.createGrid({
		Parent = statsGui.Stats.Backpack.Frame.Pockets.Slots,

		Visible = true,

		Assets = {
			Slot = script.slotAsset,
		},

		GridSize = Vector2.new(5, 3), -- How many slots the grid has on the X and Y axes.
		GridCellSize = UDim2.fromOffset(42,42),
		
		UseFrame = true,

		AnchorPoint = Vector2.new(0.5, 0.5), -- Anchor point of the grid container.
		Position = UDim2.new(0.5, 0, 0.5, 0), -- Position of the grid container.
		Size = UDim2.fromScale(1, 1), -- Size of the grid container.

		Metadata = {
			Type = "Pockets",
			MainType = "Inventory",
		},
	})	
	statsGui.Stats.Backpack.Frame.Pockets.TextLabel.Size = UDim2.fromScale(0.92*(self.grid.GridSize.X/5), 0.086)
	statsGui.Stats.Backpack.Frame.Pockets.TextLabel.Position = UDim2.fromScale(0.499*(self.grid.GridSize.X/5), 0.05)

	self.headSlot = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Equipment.Frame.Container.Head, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Head",
			MainType = "Armor",
		},
	})
	self.faceSlot = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Equipment.Frame.Container.Face, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Face",
			MainType = "Armor",
		},
	})
	self.torsoSlot = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Equipment.Frame.Container.Torso, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Torso",
			MainType = "Armor",
		},
	})
	self.legsSlot = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Equipment.Frame.Container.Legs, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Legs",
			MainType = "Armor",
		},
	})
	self.armsSlot = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Equipment.Frame.Container.Arms, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Arms",
			MainType = "Armor",
		},
	})
	self.backpackSlot = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Equipment.Frame.Container.Backpack, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Backpack",
			MainType = "Armor",
		},
	})

	self.wepSlot1 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Weapons.Frame.Container.Primary, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Primary",
			MainType = "Weapons",
			Index = 1,
		},
	})
	self.wepSlot2 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Weapons.Frame.Container.Primary2, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Primary",
			MainType = "Weapons",
			Index = 2,
		},
	})
	self.wepSlot3 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Weapons.Frame.Container.Secondary, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Secondary",
			MainType = "Weapons",
			Index = 3,
		},
	})
	self.wepSlot4 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Weapons.Frame.Container.Melee, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Melee",
			MainType = "Weapons",
			Index = 4,
		},
	})
	
	self.hotbarSlot1 = GridPack.createSingleSlot({
		Parent = self.ui.Hotbar["5"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			MainType = "Hotbar",
			Index = 5,
		},
	})
	self.hotbarSlot2 = GridPack.createSingleSlot({
		Parent = self.ui.Hotbar["6"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			MainType = "Hotbar",
			Index = 6,
		},
	})
	self.hotbarSlot3 = GridPack.createSingleSlot({
		Parent = self.ui.Hotbar["7"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			MainType = "Hotbar",
			Index = 7,
		},
	})
	self.hotbarSlot4 = GridPack.createSingleSlot({
		Parent = self.ui.Hotbar["8"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			MainType = "Hotbar",
			Index = 8,
		},
	})
	self.hotbarSlot5 = GridPack.createSingleSlot({
		Parent = self.ui.Hotbar["9"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			MainType = "Hotbar",
			Index = 9,
		},
	})
	
	self.limbSlot1 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Player.Container.Head, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Head",
			MainType = "Limbs",
		},
	})
	self.limbSlot2 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Player.Container["Left Arm"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Left Arm",
			MainType = "Limbs",
		},
	})
	self.limbSlot3 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Player.Container["Left Leg"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Left Leg",
			MainType = "Limbs",
		},
	})
	self.limbSlot4 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Player.Container["Right Arm"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Right Arm",
			MainType = "Limbs",
		},
	})
	self.limbSlot5 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Player.Container["Right Leg"], -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Right Leg",
			MainType = "Limbs",
		},
	})
	self.limbSlot6 = GridPack.createSingleSlot({
		Parent = statsGui.Stats.Player.Container.Torso, -- Parent of the slot container

		Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

		Assets = {
			Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
		},

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
		Position = UDim2.new(0, 0, 0, 0), -- Position of the slot container
		Size = UDim2.fromScale(1, 1), -- Size of the slot container

		Metadata = {
			Type = "Torso",
			MainType = "Limbs",
		},
	})

	self.transferLink = GridPack.createTransferLink({})
	self.grid:ConnectTransferLink(self.transferLink)

	self.headSlot:ConnectTransferLink(self.transferLink)
	self.torsoSlot:ConnectTransferLink(self.transferLink)
	self.legsSlot:ConnectTransferLink(self.transferLink)
	self.armsSlot:ConnectTransferLink(self.transferLink)
	self.faceSlot:ConnectTransferLink(self.transferLink)
	self.backpackSlot:ConnectTransferLink(self.transferLink)

	self.wepSlot1:ConnectTransferLink(self.transferLink)
	self.wepSlot2:ConnectTransferLink(self.transferLink)
	self.wepSlot3:ConnectTransferLink(self.transferLink)
	self.wepSlot4:ConnectTransferLink(self.transferLink)
	
	self.hotbarSlot1:ConnectTransferLink(self.transferLink)
	self.hotbarSlot2:ConnectTransferLink(self.transferLink)
	self.hotbarSlot3:ConnectTransferLink(self.transferLink)
	self.hotbarSlot4:ConnectTransferLink(self.transferLink)
	self.hotbarSlot5:ConnectTransferLink(self.transferLink)
	
	self.limbSlot1:ConnectTransferLink(self.transferLink)
	self.limbSlot2:ConnectTransferLink(self.transferLink)
	self.limbSlot3:ConnectTransferLink(self.transferLink)
	self.limbSlot4:ConnectTransferLink(self.transferLink)
	self.limbSlot5:ConnectTransferLink(self.transferLink)
	self.limbSlot6:ConnectTransferLink(self.transferLink)
	
	_G.grid = self.grid
	_G.Backpack = self.Backpack
	
	_G.wepSlot1 = self.wepSlot1
	_G.wepSlot2 = self.wepSlot2
	_G.wepSlot3 = self.wepSlot3
	_G.wepSlot4 = self.wepSlot4
	
	_G.hotbarSlot1 = self.hotbarSlot1
	_G.hotbarSlot2 = self.hotbarSlot2
	_G.hotbarSlot3 = self.hotbarSlot3
	_G.hotbarSlot4 = self.hotbarSlot4
	_G.hotbarSlot5 = self.hotbarSlot5

	return self
end

function __class:UpdateIndex(SlotData: SlotData, NewIndex: number): number
	if not NewIndex then
		local Damn = 0
		local SlotsDamn = {}
		for _, v:Instance in self.ui.Hotbar:GetChildren() do
			if not v:IsA("Frame") or v.Name == "Fist" then continue end
			Damn += 1
			SlotsDamn[v.Name] = v
		end
		for i, v in SlotsDamn do
			if SlotData.Index ~= i then continue end
			SlotData.Index = i
		end
	else
		SlotData.Index = NewIndex
	end
	return SlotData.Index
end

function __class:StatsToggle(action: string, enabled: boolean, hovering)
	if action == "enabled" then
		if enabled then
			game:GetService('Players').LocalPlayer.Character.Humanoid:UnequipTools()
			local sound:Sound = Sounds.inventory_sounds:FindFirstChild("inv_open")
			if sound then
				local clone = sound:Clone()
				clone.Parent = game:GetService('SoundService')
				clone:Play()
				game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
			end
		else
			self.ui.Parent.ActionBox.Visible = false
			self.currentAction = nil
			
			
		end

		self.statsEnabled = enabled
	end
end

function __class:AddSlot(tool: any, inInventory: boolean, imageID: string): SlotData
	if self.slots[tool] then return end
	
	for _,v in self.player.Backpack:GetChildren() do
		if v:IsA("StringValue") and v.Value == "Bullets" and v.Name == tool.Name and v ~= tool then
			if v:GetAttribute("ammo") < v:GetAttribute("max") and tool:GetAttribute("Spawned") and not tool:GetAttribute("GridRot") then
				if v:GetAttribute("max")-v:GetAttribute("ammo") < tool:GetAttribute("ammo") then
					Network:FireServer("vars", "ammoChange", {v,tool,v:GetAttribute("max")-tool:GetAttribute("ammo")})
				elseif v:GetAttribute("max")-v:GetAttribute("ammo") >= tool:GetAttribute("ammo") then
					Network:FireServer("vars", "ammoChange", {v,tool,tool:GetAttribute("ammo")})
					return
				end
			end
		end
	end

	self.slots[tool] = {
		Tool = tool,
		Trove = nil,
		Selected = false,
		Frame = nil,
		Index = -1,
		Item = nil,
		SlotFrame = nil,
		ItemManager = nil,
		hasInventory = true,
		Threads = {}
	}

	local SlotData: SlotData? = self.slots[tool]

	local Dragging = false
	local HoverFrame: Frame = nil
	local Hovered: boolean = false
	local Selected: boolean = false
	local Trove: TroveModule.TroveType = TroveModule.new()
	SlotData.Trove = Trove
	
	local curItem = nil
	
	local GUID = game:GetService("HttpService"):GenerateGUID(false)

	local invitem_temp = script.InvItem:Clone()
	--invitem_temp.Image.Image = imageID or tool.TextureId
	invitem_temp.TextFrame.ItemName.Text = tool:GetAttribute("gridName") or tool.Name

	local function Select()
		if Dragging then return end

		if self.currentTool ~= tool then
			for _, v in self.slots do
				if v.hasInventory then continue end
				v.Selected = false
			end
		elseif self.currentTool == tool then
			SlotData.Selected = false
			self.currentTool = nil
			self.toolUnequiped:Fire(tool)
			return
		end

		SlotData.Selected = not SlotData.Selected
		self.currentTool = tool

		self[SlotData.Selected and "toolEquiped" or "toolUnequiped"]:Fire(tool)
	end
	local function InputBegan(input)
		if Dragging or input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end		

		if SlotData.Selected then
			Select()
		end
	end
	
	local function renderFunc(draggingItem, hoveringItemManager, lastItemManager)
		--draggingItem = currently dragged item's UI
		--hoveringItemManager = item manager that's being hovered on

		local frame = draggingItem.ItemElement
		
		frame.UIStroke.Thickness = 3
		frame.InteractionButton.BackgroundTransparency = 0.5

		if hoveringItemManager and lastItemManager.Metadata.MainType ~= "Inventory" and lastItemManager.Metadata.Type ~= "FaceArmor" then
			local metadata = hoveringItemManager.Metadata
			if metadata and (metadata.MainType == "Inventory" or metadata.Type == "Loot") then
				frame.ViewportFrame.Visible = true
				frame.UIStroke.Enabled = true
				frame.GroupTransparency = 0

				curItem.Visible = false
			elseif metadata and metadata.MainType == lastItemManager.Metadata.MainType then
				frame.ViewportFrame.Visible = false
				frame.UIStroke.Enabled = false
				frame.GroupTransparency = 1

				curItem.Visible = true
			end
		elseif hoveringItemManager and lastItemManager.Metadata.Type == "FaceArmor" then
			local metadata = hoveringItemManager.Metadata
			if metadata and metadata.MainType == "Inventory" then
				frame.InteractionButton.BackgroundTransparency = 0.15
				frame.UIStroke.Enabled = true
			elseif metadata and metadata.MainType == lastItemManager.Metadata.MainType then
				frame.InteractionButton.BackgroundTransparency = 1
				frame.BackgroundTransparency = 1
				frame.UIStroke.Enabled = false
			end	
		end
			--[[if lastItemManager.Metadata.MainType ~= "Inventory" then
				frame.ViewportFrame.Visible = true
				frame.UIGradient.Enabled = true
				frame.UIStroke.Enabled = true
				frame.BackgroundTransparency = 0.5

				curItem.Visible = false
			end]]
	end
	local function moveFunc(movedItem, newGridPosition, newRotation, lastItemManager, newItemManager)
		--[[
            movedItem: This Item
            newGridPosition: This Item's new position in a Grid. (Doesn't apply with SingleSlots)
            newRotation: This Item's new rotation.
            lastItemManager: The ItemManager that the Item was in before it got moved.
            newItemManager: The new ItemManager the item was moved to. (If there is one)
        ]]

		local frame = movedItem.ItemElement
		
		frame.UIStroke.Thickness = 1
		frame.InteractionButton.BackgroundTransparency = 0.15

			--[[if curItem and not curItem.Visible then
				frame.ViewportFrame.Visible = false
				frame.UIGradient.Enabled = false
				frame.UIStroke.Enabled = false
				frame.BackgroundTransparency = 1

				curItem.Visible = true
			end]]

		if not newItemManager then
			if curItem ~= nil then
				frame.ViewportFrame.Visible = false
				frame.UIStroke.Enabled = false
				frame.GroupTransparency = 1

				curItem.Visible = true
			end
			
			if lastItemManager.Metadata.Type == "FaceArmor" then 
				frame.InteractionButton.BackgroundTransparency = 1
				frame.BackgroundTransparency = 1
				frame.UIStroke.Enabled = false
			end
			
			if self.storedItems[tool] then
				self.storedItems[tool] = {X = newGridPosition.X, Y = newGridPosition.Y, Rot = movedItem.PotentialRotation}
			end
			
			return 
		end
		if newItemManager.Metadata.MainType == "Armor" and (not inInventory or tool.Value == "Bullets") then return false end
		if inInventory and newItemManager.Metadata.MainType == "FaceArmor" and tool.Value ~= "FaceShield" and tool.Value ~= "NVG" then return false end
		if inInventory and newItemManager.Metadata.MainType == "FaceArmor" and tool:GetAttribute("RestrictedTo") ~= nil and tool:GetAttribute("RestrictedTo") ~= newItemManager.Metadata.Tool.Name then return false end
		
		if lastItemManager.Metadata.Type == "FaceArmor" and newItemManager.Metadata.MainType == "Limbs" then return false end		
		if newItemManager.Metadata.Type == "FaceArmor" and not newItemManager:IsColliding(movedItem, {}) then
			frame.InteractionButton.BackgroundTransparency = 1
			frame.BackgroundTransparency = 1
			frame.UIStroke.Enabled = false

			self.modifiedItem = tool
			if ArmorDB.NVG[`TIER_{tool.Tier.Value}`].itemName == tool.Name then
				Network:FireServer("vars", "NVG_Tier", {newItemManager.Metadata.Tool, tool.Tier.Value})
				if self.player.Character.Armor.Head:GetAttribute("Equipped") then
					RP.Storage.Events.PlaceNV:FireServer(tool.Tier.Value)
					self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.NVG[`TIER_{tool.Tier.Value}`].overlayImage}`

					self.ui.Parent.Parent.NVGFrame.Overlay.Visible = true
					self.ui.Parent.Parent.NVGFrame.Dust.Visible = true
				end
			else
				Network:FireServer("vars", "Face_Tier", {newItemManager.Metadata.Tool, tool.Tier.Value})
				if self.player.Character.Armor.Head:GetAttribute("Equipped") then
					RP.Storage.Events.PlaceFaceProc:FireServer(tool.Tier.Value)
					self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.FaceShield[`TIER_{tool.Tier.Value}`].overlayImage}`
				end
			end
			return true
		elseif lastItemManager.Metadata.Type == "FaceArmor" then			
			self.modifiedItem = nil
			if ArmorDB.NVG[`TIER_{tool.Tier.Value}`].itemName == tool.Name then
				Network:FireServer("vars", "NVG_Tier", {lastItemManager.Metadata.Tool, nil})
				if self.player.Character.Armor.Head:GetAttribute("Equipped") then
					RP.Storage.Events.PlaceNV:FireServer(nil)
					self.ui.Parent.Parent.NVGFrame.Overlay.Visible = false
					self.ui.Parent.Parent.NVGFrame.Dust.Visible = false
				end
			else
				Network:FireServer("vars", "Face_Tier", {lastItemManager.Metadata.Tool, nil})
				if self.player.Character.Armor.Head:GetAttribute("Equipped") then
					RP.Storage.Events.PlaceFaceProc:FireServer(nil)
					self.ui.Parent.Parent.NVGFrame.Overlay.Visible = false
				end
			end
			return true
		end
		
		if newItemManager.Metadata.Type == "Backpack" and self.Backpack then
			self.storedItems[tool] = {X = newGridPosition.X, Y = newGridPosition.Y, Rot = movedItem.PotentialRotation}
		elseif newItemManager.Metadata.Type ~= "Backpack" and self.storedItems[tool] then
			self.storedItems[tool] = nil
		end
		
		if newItemManager.Metadata.MainType == "Limbs" and tool:GetAttribute("HealItem") then
			local character = self.player.Character
			
			local canheal = false
			local current_limb = nil
			
			for _,v in pairs(character:GetChildren()) do
				if v:IsA("BasePart") and v:GetAttribute("Limbs") and v:GetAttribute("Enabled") and v.Name == newItemManager.Metadata.Type then
					local health = v:GetAttribute("Health")
					local maxhealth = v:GetAttribute("MaxHealth")
					if health >= maxhealth then continue end

					canheal = true
					current_limb = v
				end
			end
			
			if canheal then				
				character.Humanoid:EquipTool(tool)
				task.wait()				
				tool.HealLimb:Fire(newItemManager.Metadata.Type)
				_G.BPEnabled = false
			end
			return false
		elseif newItemManager.Metadata.MainType == "Limbs" and not tool:GetAttribute("HealItem") then
			return false
		end

		if newItemManager.Metadata.Type == "Loot" then
			Network:FireServer("vars", "lootSend", {tool, newItemManager.Metadata.LootName, {X = newGridPosition.X, Y = newGridPosition.Y, Rot = movedItem.PotentialRotation}})
			if lastItemManager.Metadata.MainType ~= "Inventory" and curItem then
				curItem:Destroy()
				curItem = nil
			end			
			if lastItemManager.Metadata.MainType == "Weapons" then lastItemManager.GuiElement.Parent.Icon.Visible = true end
			if lastItemManager.Metadata.MainType == "Armor" then
				game:GetService("ReplicatedStorage").Storage.Events.equipmentUse:FireServer(tool.Value, tool.Tier.Value, tool:GetAttribute("Health"), true)
				Network:FireServer("vars", "equipArmor", {tool, false})
				Network:FireServer("vars", "wearBackpack", {tool, nil})

				if tool.Value == "Backpack" then					
					self.Backpack:DisconnectTransferLink(self.transferLink)
					
					for i,v in self.storedItems do
						Network:FireServer("vars", "storeBackpackItems", {tool, i, v})
						self.Backpack:RemoveItem(self.slots[i].Item)
					end
					self.storedItems = {}

					self.Backpack:Destroy()
					self.BackpackTool = nil					

					self.statsui.Stats.Backpack.Frame.Backpack.Visible = false
				end
			end
			return
		end

		if newItemManager.Metadata.MainType == "Weapons" and not inInventory then
			local gun = tool:GetAttribute("Gun")
			local secondary = tool:GetAttribute("Secondary")
			local melee = tool:GetAttribute("Melee")

			if newItemManager.Metadata.Type == "Primary" and (not gun or secondary) then			
				return false
			end
			if newItemManager.Metadata.Type == "Secondary" and not secondary then
				return false
			end
			if newItemManager.Metadata.Type == "Melee" and not melee then
				return false
			end

			local OldFrame = SlotData.Frame

			if movedItem.Metadata.InWepSlot then
				local OldIndex = SlotData.Index
				SlotData.Index = newItemManager.Metadata.Index
				
				if not melee then Network:FireServer("vars", "weaponSlot", {tool, SlotData.Index}) end

				local ToolFrame = self.ui.Hotbar[tostring(OldIndex)]:FindFirstChild("ItemIcon")
				ToolFrame.Parent = self.ui.Hotbar[tostring(SlotData.Index)]
				ToolFrame.Position = UDim2.fromScale(0.5,0.5)
				
				lastItemManager.GuiElement.Parent.Icon.Visible = true
				newItemManager.GuiElement.Parent.Icon.Visible = false

				local random = Random.new():NextInteger(1,3)
				local sound:Sound = Sounds.inventory_sounds:FindFirstChild("move"..random)
				if sound then
					local clone = sound:Clone()
					clone.Parent = game:GetService('SoundService')
					clone:Play()
					game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
				end

				return true
			end
			
			newItemManager.GuiElement.Parent.Icon.Visible = false

			local FrameClone = script.ItemIcon:Clone()
			FrameClone.ItemName.Text = tool.Name
			FrameClone.Image = tool.TextureId

			frame.ViewportFrame.Visible = false
			frame.UIStroke.Enabled = false
			frame.GroupTransparency = 1

			curItem = script.ItemImage:Clone()
			curItem.Parent = newItemManager.GuiElement.Parent
			curItem.Image = tool.TextureId
			curItem.Size = UDim2.fromScale(0.9,0.9)

			SlotData.SlotFrame = curItem

			movedItem.Metadata.InWepSlot = true

			SlotData.hasInventory = false

			game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("inv", false)
			game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("tool", true)

			SlotData.Index = newItemManager.Metadata.Index
			
			if not melee then Network:FireServer("vars", "weaponSlot", {tool, SlotData.Index}) end

			FrameClone.Parent = self.ui.Hotbar[tostring(SlotData.Index)]
			FrameClone.Position = UDim2.fromScale(0.5,0.5)

			local random = Random.new():NextInteger(1,3)
			local sound:Sound = Sounds.inventory_sounds:FindFirstChild("move"..random)
			if sound then
				local clone = sound:Clone()
				clone.Parent = game:GetService('SoundService')
				clone:Play()
				game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
			end

			return true
		end

		if inInventory then
			if newItemManager.Metadata.MainType ~= "Armor" then
				if newItemManager.Metadata.MainType == "Weapons" or newItemManager.Metadata.MainType == "Hotbar" then return false end
				if (tool.Value == "Backpack" and lastItemManager.Metadata.MainType == "Armor") and newItemManager.Metadata.Type == "Backpack" then return false end

				if curItem ~= nil then
					curItem:Destroy()
					curItem = nil

					game:GetService("ReplicatedStorage").Storage.Events.equipmentUse:FireServer(tool.Value, tool.Tier.Value, tool:GetAttribute("Health"), true)
					Network:FireServer("vars", "equipArmor", {tool, false})
					
					if tool:GetAttribute("nvgTier") then self.ui.Parent.Parent.NVGFrame.Overlay.Visible = false self.ui.Parent.Parent.NVGFrame.Dust.Visible = false end
					if tool:GetAttribute("faceTier") then self.ui.Parent.Parent.NVGFrame.Overlay.Visible = false end
					
					Network:FireServer("vars", "wearBackpack", {tool, nil})

					frame.ViewportFrame.Visible = true
					frame.UIStroke.Enabled = true
					frame.GroupTransparency = 0
					
					if tool.Value == "Backpack" then					
						self.Backpack:DisconnectTransferLink(self.transferLink)
						for i,v in self.storedItems do
							Network:FireServer("vars", "storeBackpackItems", {tool, i, v})
							self.Backpack:RemoveItem(self.slots[i].Item)
						end
						self.storedItems = {}
						
						self.Backpack:Destroy()
						self.BackpackTool = nil
						
						self.statsui.Stats.Backpack.Frame.Backpack.Visible = false
					end
				end

				local random = Random.new():NextInteger(1,3)
				local sound:Sound = Sounds.inventory_sounds:FindFirstChild("move"..random)
				if sound then
					local clone = sound:Clone()
					clone.Parent = game:GetService('SoundService')
					clone:Play()
					game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
				end

				return true
			else
				if newItemManager.Metadata.Type ~= tool.Value then return false end

				local armorfound = newItemManager:IsColliding(movedItem, {})
				if armorfound then return false end
				
				if newItemManager.Metadata.Type == "Face" then
					if self.player.Character.Armor.Head:GetAttribute("Equipped") then
						if self.player.Character.Armor.Head:GetAttribute("usesFullHead") or self.player.Character.Armor.Head:GetAttribute("ShieldActive") then return false end
					end
				elseif newItemManager.Metadata.Type == "Head" then
					if self.player.Character.Armor.Face:GetAttribute("Equipped") then
						if self.player.Character.Armor.Head:GetAttribute("usesFullHead") or self.player.Character.Armor.Head:GetAttribute("ShieldActive") then return false end
					end
				end
			end

			local armortype = tool.Value
			local armortier = tool.Tier.Value

			frame.ViewportFrame.Visible = false
			frame.UIStroke.Enabled = false
			frame.GroupTransparency = 1

			curItem = script.ArmorImage:Clone()
			curItem.Parent = newItemManager.GuiElement.Parent
			curItem.Image = "rbxassetid://"..ArmorDB[armortype][`TIER_{armortier}`].armorImage

			SlotData.SlotFrame = curItem

			game:GetService("ReplicatedStorage").Storage.Events.equipmentUse:FireServer(armortype, armortier, tool:GetAttribute("Health"), false, tool:GetAttribute("nvgTier"), tool:GetAttribute("faceTier"))
			Network:FireServer("vars", "equipArmor", {tool, true})

			if tool:GetAttribute("nvgTier") then
				self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.NVG[`TIER_{tool:GetAttribute("nvgTier")}`].overlayImage}`
				self.ui.Parent.Parent.NVGFrame.Overlay.Visible = true
				self.ui.Parent.Parent.NVGFrame.Dust.Visible = true
			end
			if tool:GetAttribute("faceTier") then
				self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.FaceShield[`TIER_{tool:GetAttribute("faceTier")}`].overlayImage}`
			end
			
			Network:FireServer("vars", "wearBackpack", {tool, true})

			if tool.Value == "Backpack" then
				self.statsui.Stats.Backpack.Frame.Backpack.TextLabel.Text = ` {tool:GetAttribute("gridName")}`

				self.Backpack = GridPack.createGrid({
					Parent = self.statsui.Stats.Backpack.Frame.Backpack.Slots,

					Visible = true,

					Assets = {
						Slot = script.slotAsset,
					},

					GridSize = Vector2.new(tool:GetAttribute("sizeX"), tool:GetAttribute("sizeY")), -- How many slots the grid has on the X and Y axes.
					
					GridCellSize = UDim2.fromOffset(42,42),

					UseFrame = true,

					AnchorPoint = Vector2.new(0, 0), -- Anchor point of the grid container.
					Position = UDim2.new(0, 0, 0, 0), -- Position of the grid container.
					Size = UDim2.fromScale(1, 1), -- Size of the grid container.

					Metadata = {
						Type = "Backpack",
						MainType = "Inventory",
					},
				})
				self.statsui.Stats.Backpack.Frame.Backpack.TextLabel.Size = UDim2.fromScale(0.92*(self.Backpack.GridSize.X/5), 0.086)

				self.Backpack:ConnectTransferLink(self.transferLink)
				self.BackpackTool = tool
				
				if tool:GetAttribute("B_GUID") then Network:FireServer("vars", "returnBackpackItems", tool) end
				
				self.statsui.Stats.Backpack.Frame.Backpack.Visible = true
			end

			return true
		end

		if newItemManager.Metadata.MainType == "Hotbar" then
			local hasInventory = false
			local NewIndex = newItemManager.Metadata.Index

			if movedItem.Metadata.InWepSlot then
				return false
			end

			if SlotData.hasInventory then
				hasInventory = true

				local gun = tool:GetAttribute("Gun")
				local secondary = tool:GetAttribute("Secondary")
				local melee = tool:GetAttribute("Melee")
				if (gun or secondary or melee) then
					return false
				end

				if self.ui.Hotbar[tostring(NewIndex)]:FindFirstChild("ItemIcon") or tonumber(NewIndex) < 5 then 
					return false
				end

					--[[local FrameClone = script.ItemIcon:Clone()
					FrameClone.ItemName.Text = tool.Name
					FrameClone.Image = tool.TextureId

					SlotData.Frame = FrameClone]]

				frame.ViewportFrame.Visible = false
				frame.UIStroke.Enabled = false
				frame.GroupTransparency = 1

				curItem = script.ItemImage:Clone()
				curItem.Parent = newItemManager.GuiElement.Parent
				curItem.Image = tool.TextureId
				curItem.ItemName.Text = tool.Name

				SlotData.SlotFrame = curItem

				SlotData.hasInventory = false
				--self.grid:RemoveItem(movedItem)

				game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("inv", false)
				game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("tool", true)
			end

			if SlotData.Index ~= -1 then self.ui.Hotbar[tostring(SlotData.Index)].SelectedItem.Visible = false end
			SlotData.Index = tonumber(NewIndex)

			local random = Random.new():NextInteger(1,3)
			local sound:Sound = Sounds.inventory_sounds:FindFirstChild("move"..random)
			if sound then
				local clone = sound:Clone()
				clone.Parent = game:GetService('SoundService')
				clone:Play()
				game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
			end

			return
		else
			if not SlotData.hasInventory then
				local OldFrame = SlotData.Frame

				if movedItem.Metadata.InWepSlot then
					local gunicon = self.ui.Hotbar[tostring(SlotData.Index)]:FindFirstChild("ItemIcon")
					if gunicon then
						gunicon:Destroy()
					end
					if not tool:GetAttribute("Melee") then Network:FireServer("vars", "weaponSlot", {tool, nil}) end
				end
				if SlotData.Selected then Select() self.ui.Hotbar[tostring(SlotData.Index)].SelectedItem.Visible = false end

				SlotData.hasInventory = true
				movedItem.Metadata.InWepSlot = false

				SlotData.Index = -1

				frame.ViewportFrame.Visible = true
				frame.UIStroke.Enabled = true
				frame.GroupTransparency = 0

				if curItem ~= nil then
					curItem:Destroy()
					curItem = nil
					
					if lastItemManager.Metadata.MainType == "Weapons" then lastItemManager.GuiElement.Parent.Icon.Visible = true end
				end

				game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("inv", true)
				game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("tool", false)
			end

			local random = Random.new():NextInteger(1,3)
			local sound:Sound = Sounds.inventory_sounds:FindFirstChild("move"..random)
			if sound then
				local clone = sound:Clone()
				clone.Parent = game:GetService('SoundService')
				clone:Play()
				game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
			end

			return
		end

			--[[if newItemManager then
				-- Ask server to validate the Item movement between ItemManagers and return the result to the Item
				return RS.Remotes.MoveItemAcrossItemManager:InvokeServer()
			else
				-- Ask server to validate the Item movement between positions and return the result to the Item
				return RS.Remotes.MoveItem:InvokeServer()
			end]]

		-- If the result if false then the Item will move back to it's last position.
	end
	local function collideFunc(movedItem, newGridPosition, newRotation, collidedItem)
		local frame = movedItem.ItemElement
		
		frame.UIStroke.Thickness = 1
		frame.InteractionButton.BackgroundTransparency = 0.15
		
		if inInventory and tool.Value == "Bullets" then
			local foundtool = collidedItem.Metadata.Tool
			if foundtool.Name == tool.Name then
				if foundtool:GetAttribute("ammo") < foundtool:GetAttribute("max") then
					if foundtool:GetAttribute("max")-foundtool:GetAttribute("ammo") < tool:GetAttribute("ammo") then
						Network:FireServer("vars", "ammoChange", {foundtool,tool,foundtool:GetAttribute("max")-foundtool:GetAttribute("ammo")})
					elseif foundtool:GetAttribute("max")-foundtool:GetAttribute("ammo") >= tool:GetAttribute("ammo") then
						Network:FireServer("vars", "ammoChange", {foundtool,tool,tool:GetAttribute("ammo")})
					end
				end
			end
		else
			if curItem ~= nil then
				frame.ViewportFrame.Visible = false
				frame.UIStroke.Enabled = false
				frame.GroupTransparency = 1

				curItem.Visible = true
			end
			if movedItem.ItemManager.Metadata.Type == "FaceArmor" then 
				frame.InteractionButton.BackgroundTransparency = 1
				frame.BackgroundTransparency = 1
				frame.UIStroke.Enabled = false
			end
		end
	end

	local myFirstItem = GridPack.createItem({
		Position = Vector2.new(tool:GetAttribute("GridPosX") or 0, tool:GetAttribute("GridPosY") or 0),
		Size = WeaponSizes[tool:GetAttribute("InvSize")] or Vector2.new(2, 3),
		
		Rotation = tool:GetAttribute("GridRot") or 0,

		Assets = {
			Item = invitem_temp,
		},

		MoveMiddleware = moveFunc,
		
		RenderMiddleware = renderFunc,
		
		ClickMiddleware = function(clickedItem, mousePos)	
			if not self.ui.Parent.ActionBox.Visible then self.ui.Parent.ActionBox.Visible = true end

			self.currentAction = clickedItem

			local frameSize = self.ui.Parent.ActionBox.AbsoluteSize
			self.ui.Parent.ActionBox.Position = UDim2.fromOffset(mousePos.X - frameSize.X, mousePos.Y + (frameSize.Y*1.75))
			
			if inInventory then
				self.ui.Parent.ActionBox.Modify.Visible = tool:GetAttribute("canModify")
			end
		end,
		
		CollideMiddleware = collideFunc,

		Metadata = {
			InWepSlot = false,
			Tool = tool,
		},
	})
	if not tool:GetAttribute("wepSlot") then
		if not tool:GetAttribute("armorSlot") then
			if not tool:GetAttribute("hotbarSlot") then
				if tool:GetAttribute("GridType") == "Backpack" then
					self.Backpack:AddItem(myFirstItem, nil, not tool:GetAttribute("GridRot"))
					self.storedItems[tool] = {X = myFirstItem.Position.X, Y = myFirstItem.Position.Y, Rot = myFirstItem.Rotation}
				else
					if tool:GetAttribute("GridType") == "FaceArmor" then
						if self.player.Character.Armor.Head:GetAttribute("Equipped") then
							if ArmorDB.NVG[`TIER_{tool.Tier.Value}`].itemName == tool.Name then
								Network:FireServer("vars", "NVG_Tier", {self.player.Backpack:FindFirstChild(ArmorDB.Head[`TIER_{self.player.Character.Armor.Head.Tier.Value}`].itemName), tool.Tier.Value})

								RP.Storage.Events.PlaceNV:FireServer(tool.Tier.Value)
								self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.NVG[`TIER_{tool.Tier.Value}`].overlayImage}`

								self.ui.Parent.Parent.NVGFrame.Overlay.Visible = true
								self.ui.Parent.Parent.NVGFrame.Dust.Visible = true
								
								myFirstItem:Destroy()
								Network:FireServer("vars", "storeModifiedItems", {self.player.Backpack:FindFirstChild(ArmorDB.Head[`TIER_{self.player.Character.Armor.Head.Tier.Value}`].itemName), tool})
							else
								if not tool:GetAttribute("RestrictedTo") or (tool:GetAttribute("RestrictedTo") and tool:GetAttribute("RestrictedTo") == ArmorDB.Head[`TIER_{self.player.Character.Armor.Head.Tier.Value}`].itemName) then
									Network:FireServer("vars", "Face_Tier", {self.player.Backpack:FindFirstChild(ArmorDB.Head[`TIER_{self.player.Character.Armor.Head.Tier.Value}`].itemName), tool.Tier.Value})

									RP.Storage.Events.PlaceFaceProc:FireServer(tool.Tier.Value)
									self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.FaceShield[`TIER_{tool.Tier.Value}`].overlayImage}`

									myFirstItem:Destroy()
									Network:FireServer("vars", "storeModifiedItems", {self.player.Backpack:FindFirstChild(ArmorDB.Head[`TIER_{self.player.Character.Armor.Head.Tier.Value}`].itemName), tool})
								else
									self.grid:AddItem(myFirstItem, nil, not tool:GetAttribute("GridRot"))
								end
							end
						else
							self.grid:AddItem(myFirstItem, nil, not tool:GetAttribute("GridRot"))
						end
					else
						if tool:GetAttribute("GridType") == "Modify" then
							myFirstItem.ItemElement.InteractionButton.BackgroundTransparency = 1
							myFirstItem.ItemElement.BackgroundTransparency = 1
							myFirstItem.ItemElement.UIStroke.Enabled = false

							self.modifiedItem = tool
							self.modifySlot:ChangeItem(myFirstItem)
							
							if ArmorDB.NVG[`TIER_{tool.Tier.Value}`].itemName == tool.Name then
								Network:FireServer("vars", "NVG_Tier", {self.modifySlot.Metadata.Tool, tool.Tier.Value})
							else
								Network:FireServer("vars", "Face_Tier", {self.modifySlot.Metadata.Tool, tool.Tier.Value})
							end
						else
							self.grid:AddItem(myFirstItem, nil, not tool:GetAttribute("GridRot"))
						end
					end
				end
			else
				self[`hotbarSlot{tool:GetAttribute("hotbarSlot")}`]:ChangeItem(myFirstItem)
				
				myFirstItem.ItemElement.ViewportFrame.Visible = false
				myFirstItem.ItemElement.UIStroke.Enabled = false
				myFirstItem.ItemElement.GroupTransparency = 1

				curItem = script.ItemImage:Clone()
				curItem.Parent = myFirstItem.ItemManager.GuiElement.Parent
				curItem.Image = tool.TextureId
				curItem.ItemName.Text = tool.Name

				SlotData.SlotFrame = curItem

				SlotData.hasInventory = false

				SlotData.Index = myFirstItem.ItemManager.Metadata.Index

				local random = Random.new():NextInteger(1,3)
				local sound:Sound = Sounds.inventory_sounds:FindFirstChild("move"..random)
				if sound then
					local clone = sound:Clone()
					clone.Parent = game:GetService('SoundService')
					clone:Play()
					game:GetService("Debris"):AddItem(clone, (sound.TimeLength/sound.PlaybackSpeed)+0.05)
				end
			end
		else
			self[`{string.lower(tool.Value)}Slot`]:ChangeItem(myFirstItem)
			
			local armortype = tool.Value
			local armortier = tool.Tier.Value

			myFirstItem.ItemElement.ViewportFrame.Visible = false
			myFirstItem.ItemElement.UIStroke.Enabled = false
			myFirstItem.ItemElement.GroupTransparency = 1

			curItem = script.ArmorImage:Clone()
			curItem.Parent = myFirstItem.ItemManager.GuiElement.Parent
			curItem.Image = "rbxassetid://"..ArmorDB[armortype][`TIER_{armortier}`].armorImage

			SlotData.SlotFrame = curItem

			game:GetService("ReplicatedStorage").Storage.Events.equipmentUse:FireServer(armortype, armortier, tool:GetAttribute("Health"), false, tool:GetAttribute("nvgTier"), tool:GetAttribute("faceTier"))
			Network:FireServer("vars", "equipArmor", {tool, true})
			
			if tool:GetAttribute("nvgTier") then
				self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.NVG[`TIER_{tool:GetAttribute("nvgTier")}`].overlayImage}`
				self.ui.Parent.Parent.NVGFrame.Overlay.Visible = true
				self.ui.Parent.Parent.NVGFrame.Dust.Visible = true
			end
			if tool:GetAttribute("faceTier") then
				self.ui.Parent.Parent.NVGFrame.Overlay.Image = `rbxassetid://{ArmorDB.FaceShield[`TIER_{tool:GetAttribute("faceTier")}`].overlayImage}`
			end
			
			Network:FireServer("vars", "wearBackpack", {tool, true})

			if tool.Value == "Backpack" then
				self.statsui.Stats.Backpack.Frame.Backpack.TextLabel.Text = ` {tool:GetAttribute("gridName")}`

				self.Backpack = GridPack.createGrid({
					Parent = self.statsui.Stats.Backpack.Frame.Backpack.Slots,

					Visible = true,

					Assets = {
						Slot = script.slotAsset,
					},

					GridSize = Vector2.new(tool:GetAttribute("sizeX"), tool:GetAttribute("sizeY")), -- How many slots the grid has on the X and Y axes.

					GridCellSize = UDim2.fromOffset(42,42),

					UseFrame = true,

					AnchorPoint = Vector2.new(0, 0), -- Anchor point of the grid container.
					Position = UDim2.new(0, 0, 0, 0), -- Position of the grid container.
					Size = UDim2.fromScale(1, 1), -- Size of the grid container.

					Metadata = {
						Type = "Backpack",
						MainType = "Inventory",
					},
				})
				self.statsui.Stats.Backpack.Frame.Backpack.TextLabel.Size = UDim2.fromScale(0.92*(self.Backpack.GridSize.X/5), 0.086)

				self.Backpack:ConnectTransferLink(self.transferLink)
				self.BackpackTool = tool
				
				if tool:GetAttribute("B_GUID") then Network:FireServer("vars", "returnBackpackItems", tool) end
				
				self.statsui.Stats.Backpack.Frame.Backpack.Visible = true
			end
		end
	else
		self[`wepSlot{tool:GetAttribute("wepSlot")}`]:ChangeItem(myFirstItem)
		
		myFirstItem.ItemManager.GuiElement.Parent.Icon.Visible = false

		local FrameClone = script.ItemIcon:Clone()
		FrameClone.ItemName.Text = tool.Name
		FrameClone.Image = tool.TextureId

		myFirstItem.ItemElement.ViewportFrame.Visible = false
		myFirstItem.ItemElement.UIStroke.Enabled = false
		myFirstItem.ItemElement.GroupTransparency = 1

		curItem = script.ItemImage:Clone()
		curItem.Parent = myFirstItem.ItemManager.GuiElement.Parent
		curItem.Image = tool.TextureId
		curItem.Size = UDim2.fromScale(0.9,0.9)

		SlotData.SlotFrame = curItem

		myFirstItem.Metadata.InWepSlot = true

		SlotData.hasInventory = false

		SlotData.Index = myFirstItem.ItemManager.Metadata.Index

		Network:FireServer("vars", "weaponSlot", {tool, SlotData.Index})

		FrameClone.Parent = self.ui.Hotbar[tostring(SlotData.Index)]
		FrameClone.Position = UDim2.fromScale(0.5,0.5)
	end
	task.delay(0.25, function()
		Network:FireServer("vars", "removeGridAtts", tool)
	end)
	
	SlotData.Item = myFirstItem
	local viewsettings = WeaponView[tool.Name]
	
	local cam = Instance.new("Camera")
	cam.FieldOfView = typeof(viewsettings) == "table" and viewsettings[2] or 70
	cam.Parent = myFirstItem.ItemElement.ViewportFrame

	myFirstItem.ItemElement.ViewportFrame.CurrentCamera = cam

	local mod = Instance.new("Model", myFirstItem.ItemElement.ViewportFrame)
	
	if not inInventory then
		repeat task.wait() until tool:GetAttribute("modelName") and RP.Storage.Models.Weapons:FindFirstChild(tool:GetAttribute("modelName")) or RP.Storage.Models.Weapons:FindFirstChild(tool.Name)
		local model = tool:GetAttribute("modelName") and RP.Storage.Models.Weapons:FindFirstChild(tool:GetAttribute("modelName")) or RP.Storage.Models.Weapons:FindFirstChild(tool.Name)
		model = model:Clone()
		model.Parent = mod
	else
		if tool:FindFirstChild("ArmorName") then
			repeat task.wait() until RP.Storage.Models.Armors:FindFirstChild(tool.Value):FindFirstChild("TIER_"..tool.Tier.Value)
			local model = RP.Storage.Models.Armors:FindFirstChild(tool.Value):FindFirstChild("TIER_"..tool.Tier.Value).Models
			model = model:Clone()
			model.Parent = mod

			if tool:GetAttribute("nvgTier") then
				local visor = nil
				for _,v in model:GetDescendants() do
					if v:IsA("BasePart") then
						if v.Name == "Visor" then
							visor = v
							break
						end
					end
				end

				local nv_model = RP.Storage.Models.Armors.NVG[`TIER_{tool:GetAttribute("nvgTier")}`].Models
				nv_model = nv_model:Clone()
				nv_model.Name = "NV_Models"
				nv_model:FindFirstChildOfClass("Model"):PivotTo(visor.CFrame)
				nv_model.Parent = mod
			elseif tool:GetAttribute("faceTier") then
				for _,v in model:GetDescendants() do
					if v:IsA("BasePart") then
						if v.Name == "ShieldHandle" then
							for _,h in v.Parent.Close:GetChildren() do
								if h:IsA("BasePart") then
									h.Transparency = h:GetAttribute("OldTransparency") or 0
								end
							end
						end
					end
				end
			end
		elseif tool.Value == "Bullets" then
			repeat task.wait() until tool:GetAttribute("modelName") and RP.Storage.Models.Cartridges:FindFirstChild(tool:GetAttribute("modelName")) or RP.Storage.Models.Cartridges:FindFirstChild(tool.Name)
			local model = tool:GetAttribute("modelName") and RP.Storage.Models.Cartridges:FindFirstChild(tool:GetAttribute("modelName")) or RP.Storage.Models.Cartridges:FindFirstChild(tool.Name)
			model = model:Clone()
			model.Parent = mod
		end
	end
	
	Trove:Connect(tool.AttributeChanged, function()
		if tool:GetAttribute("nvgTier") then
			if mod:FindFirstChild("NV_Models") then return end

			local visor = nil
			for _,v in mod:GetDescendants() do
				if v:IsA("BasePart") then
					if v.Name == "Visor" then
						visor = v
						break
					end
				end
			end

			local nv_model = RP.Storage.Models.Armors.NVG[`TIER_{tool:GetAttribute("nvgTier")}`].Models
			nv_model = nv_model:Clone()
			nv_model.Name = "NV_Models"
			nv_model:FindFirstChildOfClass("Model"):PivotTo(visor.CFrame)
			nv_model.Parent = mod
		elseif not tool:GetAttribute("nvgTier") then
			if mod:FindFirstChild("NV_Models") then mod.NV_Models:Destroy() end
		end
		
		if mod:FindFirstChild("Models") and not mod.Models:FindFirstChildOfClass("Model"):FindFirstChild("ShieldHandle") then return end
		
		if tool:GetAttribute("faceTier") then
			for _,v in mod:GetDescendants() do
				if v:IsA("BasePart") then
					if v.Name == "ShieldHandle" then
						for _,h in v.Parent.Close:GetChildren() do
							if h:IsA("BasePart") then
								h.Transparency = h:GetAttribute("OldTransparency") or 0
							end
						end
					end
				end
			end
		elseif not tool:GetAttribute("faceTier") then
			for _,v in mod:GetDescendants() do
				if v:IsA("BasePart") then
					if v.Name == "ShieldHandle" then
						for _,h in v.Parent.Close:GetChildren() do
							if h:IsA("BasePart") then
								h.Transparency = 1
							end
						end
					end
				end
			end
		end
	end)
	
	for i, v in Rarity do
		for i1, v1 in v.Items do
			if v1 == tool.Name then
				myFirstItem.ItemElement.InteractionButton.BackgroundColor3 = v.Color
				myFirstItem.ItemElement.ViewportFrame.BackgroundColor3 = v.Color
			end
		end
	end

	--[[local cf = cam.CFrame+(cam.CFrame.LookVector*1)
	cf *= WeaponView[tool.Name] or CFrame.identity
	
	mod:PivotTo(cf)]]

	local vpfModel = ViewportModel.new(myFirstItem.ItemElement.ViewportFrame, cam)
	local cf, size = mod:GetBoundingBox()

	vpfModel:SetModel(mod)

	--[[local distance = vpfModel:GetFitDistance(cf.Position)
	cam.CFrame = CFrame.new(cf.Position) * CFrame.new(0, 0, distance)]]
	local orientation = CFrame.fromEulerAnglesYXZ(0, 0, 0)
	cam.CFrame = (typeof(viewsettings) == "table" and viewsettings[1] or viewsettings) or vpfModel:GetMinimumFitCFrame(orientation)
	--cam.CFrame *= WeaponView[tool.Name] or CFrame.identity

	local Frame = myFirstItem.ItemElement
	SlotData.Frame = Frame

	game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("inv", true)
	
	myFirstItem.ItemElement.Fade.BackgroundTransparency = 0
	TS:Create(myFirstItem.ItemElement.Fade, TweenInfo.new(0.15), {BackgroundTransparency = 1}):Play()

	Trove:Connect(UIS.InputBegan, function(input, gpe)
		if Dragging or gpe or not self.enabled then return end

		for i, v in tfSlots do
			if input.KeyCode ~= v or i ~= SlotData.Index or SlotData.Index == -1 then continue end
			Select()
		end
	end)

	Trove:Connect(tool.AncestryChanged, function()
		if not self.player and not self.player.Backpack then return end
		if tool:IsDescendantOf(self.player.Backpack) and SlotData.Selected then
			Select()
		end
	end)
	
	Trove:Connect(myFirstItem.ItemElement.InteractionButton.MouseButton1Down, function()
		if not self.enabled or self.currentAction ~= myFirstItem then return end
		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)

	Trove:Connect(self.ui.Parent.ActionBox.Inspect.MouseButton1Click, function()
		if not self.enabled or self.currentAction ~= myFirstItem then return end
		if self.ui.Parent:FindFirstChild(`hover_{GUID}`) then return end
	--	print(`inspect {tool.Name} pls`)

		local hoverinfo = self.player.PlayerGui.UI.HUD_GAMEPLAY.HoverInfo:Clone()
		hoverinfo.Parent = self.player.PlayerGui.UI.HUD_GAMEPLAY
		hoverinfo.Name = `hover_{GUID}`
		hoverinfo.Visible = true

		local viewportModelRotator = require(game:GetService('ReplicatedStorage').ViewportRotation)

		hoverinfo.Frame.ItemName.Frame.SettingsClose.MouseButton1Click:Connect(function()
			viewportModelRotator:Disable(hoverinfo.Frame.Frame.ViewportFrame)
			game:GetService("Debris"):AddItem(hoverinfo, 1/99)
			--task.wait()
			--hoverinfo:Destroy()
		end)	
		self.statsui:GetPropertyChangedSignal("Visible"):Once(function()
			print(hoverinfo:GetChildren())
			viewportModelRotator:Disable(hoverinfo.Frame.Frame.ViewportFrame)
			game:GetService("Debris"):AddItem(hoverinfo, 1/99)
			--task.wait()
			--hoverinfo:Destroy()
		end)

		local config = (RP.Storage.Modules.Weapons:FindFirstChild(tool.Name) and require(RP.Storage.Modules.Weapons:FindFirstChild(tool.Name))) or (RP.Storage.Modules.Magazines:FindFirstChild(tool.Name) and require(RP.Storage.Modules.Magazines:FindFirstChild(tool.Name))) or nil
		local mouse:Mouse = self.player:GetMouse()

		local frameSize = hoverinfo.AbsoluteSize
		hoverinfo.Position = UDim2.fromOffset(mouse.X+(frameSize.X*0.5), mouse.Y+(frameSize.Y*0.65))

		hoverinfo.Frame.Frame.ItemInfo.TextLabel.Text = inInventory and ((tool:FindFirstChild("ArmorInfo") and tool.ArmorInfo.Value) or (tool:FindFirstChild("CaliberInfo") and tool.CaliberInfo.Value) or "") or ((config and config.description) and config.description or "")
		hoverinfo.Frame.ItemName.Frame.ItemText.Text = tool:FindFirstChild("ArmorName") and tool.ArmorName.Value or tool.Name

		local templateframe = hoverinfo.Frame.Frame.StatsInfo.Main.template:FindFirstChild("templateFrame")

		local names = {
			["rate"] = "fire rate",
			["mode"] = "types of fire",
			["vel"] = "muzzle velocity",
		}
		local firemodes = {
			"single fire",
			"burst fire",
			"full auto",
		}
		local firemodes_min = {
			"single",
			"burst",
			"auto",
		}

		if config then
			for i,v in config do
				if tool:GetAttribute("Gun") then
					local name = tostring(i) 
					if name == "damage" or name == "rate" or name == "mode" or name == "vel" or name == "recoil" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						if typeof(v) == "table" and name == "damage" then					
							frame.StatName.Text = string.upper(name)
							frame.StatValue.Text = v[1] == v[2] and `{v[1]}` or `{v[1]}~{v[2]}`
							frame.Bar.Size = UDim2.fromScale(math.clamp(v[2]/100, 0, 1), 0.9)
						elseif typeof(v) == "table" and name == "recoil" then
							frame.StatName.Text = string.upper("horizontal recoil")
							frame.StatValue.Text = math.abs(v.AngleY_Min) == v.AngleY_Max and `{math.floor(v.AngleY_Max)}` or `{math.abs(math.floor(v.AngleY_Min))}~{math.floor(v.AngleY_Max)}`
							frame.Bar.Size = UDim2.fromScale(math.clamp(v.AngleY_Max/5, 0, 1), 0.9)

							local frame2 = templateframe:Clone()
							frame2.Name = `{name}2`
							frame2.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

							frame2.StatName.Text = string.upper("vertical recoil")
							frame2.StatValue.Text = math.abs(v.AngleX_Min) == v.AngleX_Max and `{math.floor(v.AngleX_Max)}` or `{math.abs(math.floor(v.AngleX_Min))}~{math.floor(v.AngleX_Max)}`
							frame2.Bar.Size = UDim2.fromScale(math.clamp(v.AngleX_Max/30, 0, 1), 0.9)

							frame2.Visible = true
						elseif typeof(v) == "number" then
							local hasswitch = name == "mode" and config.mode_switch
							local available_modes = hasswitch and (config.mode_switch[3] and `{firemodes_min[config.mode_switch[1]]}/{firemodes_min[config.mode_switch[2]]}/{firemodes_min[config.mode_switch[3]]}` or `{firemodes_min[config.mode_switch[1]]}/{firemodes_min[config.mode_switch[2]]}`) or nil
							
							frame.StatName.Text = string.upper(names[name])
							frame.StatValue.Text = (name == "mode" and (hasswitch and string.upper(available_modes) or `{string.upper(firemodes[v])}`)) or (name == "rate" and `~{v}s`) or `~{v} m/s`
							frame.Bar.Visible = false
						end

						frame.Visible = true
					end
				elseif tool:GetAttribute("Melee") then
					local name = tostring(i) 
					if name == "damage" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper(name)
						frame.StatValue.Text = v[1] == v[2] and `{v[1]}` or `{v[1]}~{v[2]}`
						frame.Bar.Size = UDim2.fromScale(math.clamp(v[2]/100, 0, 1), 0.9)

						frame.Visible = true
					end
				elseif tool:GetAttribute("HealItem") then
					local name = tostring(i) 
					if name == "maxuses" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper("heal unit")
						frame.StatValue.Text = v
						frame.Bar.Visible = false

						frame.Visible = true
					end
				elseif tool:GetAttribute("Magazine") then
					local name = tostring(i) 
					if name == "max" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper("magazine capacity")
						frame.StatValue.Text = v
						frame.Bar.Visible = false

						frame.Visible = true
					elseif name == "caliber_name" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper("caliber")
						frame.StatValue.Text = v
						frame.Bar.Visible = false

						frame.Visible = true
					end
				end
			end
		end

		if inInventory then
			if tool:GetAttribute("canSlide") ~= nil then
				local frame = templateframe:Clone()
				frame.Name = "canslide"
				frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

				frame.StatName.Text = string.upper("can slide")
				frame.StatValue.Text = tool:GetAttribute("canSlide") and "YES" or "NO"
				frame.Bar.Visible = false

				frame.Visible = true
			end
			if tool:GetAttribute("sizeX") ~= nil and tool:GetAttribute("sizeY") ~= nil then
				local frame = templateframe:Clone()
				frame.Name = "capacity"
				frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

				frame.StatName.Text = string.upper("capacity")
				frame.StatValue.Text = `{tool:GetAttribute("sizeX")*tool:GetAttribute("sizeY")}`
				frame.Bar.Size = UDim2.fromScale(math.clamp((tool:GetAttribute("sizeX")*tool:GetAttribute("sizeY"))/100, 0, 1), 0.9)

				frame.Visible = true
			end
			if tool:GetAttribute("MaxHealth") > 5 then
				local frame = templateframe:Clone()
				frame.Name = "durability"
				frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

				frame.StatName.Text = string.upper("durability")
				frame.StatValue.Text = `{tool:GetAttribute("MaxHealth")}`
				frame.Bar.Visible = false

				frame.Visible = true
			end
		end

		local mod = Instance.new("Model", hoverinfo.Frame.Frame.ViewportFrame)
		if not inInventory then
			local model = tool:GetAttribute("modelName") and RP.Storage.Models.Weapons:FindFirstChild(tool:GetAttribute("modelName")) or RP.Storage.Models.Weapons:FindFirstChild(tool.Name)
			model = model:Clone()
			model.Parent = mod

			if model:GetAttribute("useY") then mod:SetAttribute("useY", true) end
			if model:GetAttribute("inverseY") then mod:SetAttribute("inverseY", true) end
			if model:GetAttribute("bothY") then mod:SetAttribute("bothY", true) end
		else
			if tool.Value ~= "Bullets" then
				local model = RP.Storage.Models.Armors:FindFirstChild(tool.Value):FindFirstChild("TIER_"..tool.Tier.Value).Models
				model = model:Clone()
				model.Parent = mod

				if model:GetAttribute("useY") then mod:SetAttribute("useY", true) end
				if model:GetAttribute("inverseY") then mod:SetAttribute("inverseY", true) end
				if model:GetAttribute("bothY") then mod:SetAttribute("bothY", true) end
			else
				local model = tool:GetAttribute("modelName") and RP.Storage.Models.Cartridges:FindFirstChild(tool:GetAttribute("modelName")) or RP.Storage.Models.Cartridges:FindFirstChild(tool.Name)
				model = model:Clone()
				model.Parent = mod

				if model:GetAttribute("useY") then mod:SetAttribute("useY", true) end
				if model:GetAttribute("inverseY") then mod:SetAttribute("inverseY", true) end
				if model:GetAttribute("bothY") then mod:SetAttribute("bothY", true) end
			end
		end

		local cam = Instance.new("Camera")
		cam.Parent = hoverinfo.Frame.Frame.ViewportFrame

		hoverinfo.Frame.Frame.ViewportFrame.CurrentCamera = cam	
		cam.CFrame = (typeof(viewsettings) == "table" and viewsettings[1] or viewsettings) or vpfModel:GetMinimumFitCFrame(orientation)

		local pos = mod:GetBoundingBox()

		local part = Instance.new("Part", mod)
		part.Name = "RotPart"
		part.Size = Vector3.new(0.05,0.05,0.05)
		part.Transparency = 1
		part.CanCollide = false
		part.CFrame = pos

		mod.PrimaryPart = part

		viewportModelRotator:Enable(hoverinfo.Frame.Frame.ViewportFrame,3.5)

		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)
	Trove:Connect(self.ui.Parent.ActionBox.Modify.MouseButton1Click, function()
		if not self.enabled or self.currentAction ~= myFirstItem then return end
		if self.player.PlayerGui.UI.HUD_GAMEPLAY.ModifyInfo.Visible then return end
		if not inInventory then return end
		print(`modify {tool.Name} pls`)

		local hoverinfo = self.player.PlayerGui.UI.HUD_GAMEPLAY.ModifyInfo
		hoverinfo.Visible = true

		local viewportModelRotator = require(game:GetService('ReplicatedStorage').ViewportRotation)

		local modifySlot = GridPack.createSingleSlot({
			Parent = hoverinfo.Frame.Frame.StatsInfo.Main.modifyFrame, -- Parent of the slot container

			Visible = true, -- If the slot is visible, changes the containers visible property. Also disables item interaction on the item inside.

			Assets = {
				Slot = script.singleSlot, -- Add your own CanvasGroup here to customize the slot.
			},

			AnchorPoint = Vector2.new(0, 0), -- Anchor point of the slot container
			Position = UDim2.fromScale(0,0), -- Position of the slot container
			Size = UDim2.fromScale(1.3, 1.3), -- Size of the slot container

			Metadata = {
				Type = "FaceArmor",
				MainType = "Armor",
				Tool = tool,
			},
		})
		modifySlot:ConnectTransferLink(self.transferLink)

		self.modifySlot = modifySlot

		if tool:GetAttribute("G_GUID") then Network:FireServer("vars", "returnModifiedItems", tool) end

		local attribute_con = nil

		hoverinfo.Frame.ItemName.Frame.SettingsClose.MouseButton1Click:Connect(function()
			attribute_con:Disconnect()
			viewportModelRotator:Disable(hoverinfo.Frame.Frame.ViewportFrame)
			for _,v in hoverinfo.Frame.Frame.ViewportFrame:GetChildren() do
				if not v:IsA("UIStroke") then v:Destroy() end
			end
			modifySlot:DisconnectTransferLink(self.transferLink)
			if self.modifiedItem then
				modifySlot:RemoveItem(self.slots[self.modifiedItem].Item)
				Network:FireServer("vars", "storeModifiedItems", {tool, self.modifiedItem})
			end
			task.wait()
			modifySlot:Destroy()
			self.modifiedItem = nil
			hoverinfo.Visible = false
		end)		
		self.statsui:GetPropertyChangedSignal("Visible"):Once(function()
			attribute_con:Disconnect()
			viewportModelRotator:Disable(hoverinfo.Frame.Frame.ViewportFrame)
			for _,v in hoverinfo.Frame.Frame.ViewportFrame:GetChildren() do
				if not v:IsA("UIStroke") then v:Destroy() end
			end
			modifySlot:DisconnectTransferLink(self.transferLink)
			if self.modifiedItem then
				modifySlot:RemoveItem(self.slots[self.modifiedItem].Item)
				Network:FireServer("vars", "storeModifiedItems", {tool, self.modifiedItem})
			end
			task.wait()
			modifySlot:Destroy()
			self.modifiedItem = nil
			hoverinfo.Visible = false
		end)

		local mouse:Mouse = self.player:GetMouse()

		local frameSize = hoverinfo.AbsoluteSize
		hoverinfo.Position = UDim2.fromOffset(mouse.X+(frameSize.X*0.5), mouse.Y+(frameSize.Y*0.65))

		hoverinfo.Frame.ItemName.Frame.ItemText.Text = tool:FindFirstChild("ArmorName") and tool.ArmorName.Value or tool.Name

		local mod = Instance.new("Model", hoverinfo.Frame.Frame.ViewportFrame)

		local model = RP.Storage.Models.Armors:FindFirstChild(tool.Value):FindFirstChild("TIER_"..tool.Tier.Value).Models
		model = model:Clone()
		model.Parent = mod

		if model:GetAttribute("useY") then mod:SetAttribute("useY", true) end
		if model:GetAttribute("inverseY") then mod:SetAttribute("inverseY", true) end
		if model:GetAttribute("bothY") then mod:SetAttribute("bothY", true) end

		attribute_con = tool.AttributeChanged:Connect(function()
			if tool:GetAttribute("nvgTier") then
				if mod:FindFirstChild("NV_Models") then return end

				local visor = nil
				for _,v in model:GetDescendants() do
					if v:IsA("BasePart") then
						if v.Name == "Visor" then
							visor = v
							break
						end
					end
				end

				local nv_model = RP.Storage.Models.Armors.NVG[`TIER_{tool:GetAttribute("nvgTier")}`].Models
				nv_model = nv_model:Clone()
				nv_model.Name = "NV_Models"
				nv_model:FindFirstChildOfClass("Model"):PivotTo(visor.CFrame)
				nv_model.Parent = mod
			elseif not tool:GetAttribute("nvgTier") then
				if mod:FindFirstChild("NV_Models") then mod.NV_Models:Destroy() end
			end
			
			if not mod.Models:FindFirstChildOfClass("Model"):FindFirstChild("ShieldHandle") then return end

			if tool:GetAttribute("faceTier") then
				for _,v in mod:GetDescendants() do
					if v:IsA("BasePart") then
						if v.Name == "ShieldHandle" then
							for _,h in v.Parent.Close:GetChildren() do
								if h:IsA("BasePart") then
									h.Transparency = h:GetAttribute("OldTransparency") or 0
								end
							end
						end
					end
				end
			elseif not tool:GetAttribute("faceTier") then
				for _,v in mod:GetDescendants() do
					if v:IsA("BasePart") then
						if v.Name == "ShieldHandle" then
							for _,h in v.Parent.Close:GetChildren() do
								if h:IsA("BasePart") then
									h.Transparency = 1
								end
							end
						end
					end
				end
			end
		end)

		local cam = Instance.new("Camera")
		cam.Parent = hoverinfo.Frame.Frame.ViewportFrame

		hoverinfo.Frame.Frame.ViewportFrame.CurrentCamera = cam	
		cam.CFrame = (typeof(viewsettings) == "table" and viewsettings[1] or viewsettings) or vpfModel:GetMinimumFitCFrame(orientation)

		local pos = mod:GetBoundingBox()

		local part = Instance.new("Part", mod)
		part.Name = "RotPart"
		part.Size = Vector3.new(0.05,0.05,0.05)
		part.Transparency = 1
		part.CanCollide = false
		part.CFrame = pos

		mod.PrimaryPart = part

		viewportModelRotator:Enable(hoverinfo.Frame.Frame.ViewportFrame,3.5)

		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)
	Trove:Connect(self.ui.Parent.ActionBox.Discard.MouseButton1Click, function()
		if not self.enabled or self.currentAction ~= myFirstItem then return end

		if tool:GetAttribute("CantDrop") or inInventory then
			_G.messageLog({
				Text = "YOU CAN'T DROP THIS ITEM",
				TextColor3 = Color3.fromRGB(255, 94, 97),
			}, false, 2)
		else
			Network:FireServer("pass", tool, self.player.Character.HumanoidRootPart.Velocity + Vector3.new(0,20,0))
		end

		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)

	Trove:Connect(RS.RenderStepped, function()
		if SlotData.Selected and not Selected then
			Selected = true
		elseif not SlotData.Selected and Selected then
			Selected = false
		end

		if not SlotData.hasInventory then
			if self.ui.Hotbar[tostring(SlotData.Index)]:FindFirstChild("SelectedItem") then
				self.ui.Hotbar[tostring(SlotData.Index)].SelectedItem.Visible = Selected
			end
			if curItem then curItem.Position = myFirstItem.ItemElement.Position+UDim2.fromScale(0.5,0.5) curItem.Parent = myFirstItem.ItemElement.Parent end
		else
			if curItem and inInventory then curItem.health.Text = `{math.floor(tool:GetAttribute("Health"))}/{ArmorDB[tool.Value][`TIER_{tool.Tier.Value}`].durability}` curItem.health.TextColor3 = tool:GetAttribute("Health") <= 0 and Color3.fromRGB(147, 6, 2):Lerp(Color3.fromRGB(200, 65, 67), math.abs(math.sin(tick() * 2.5) * 0.5)) or Color3.fromRGB(255,255,255) curItem.Position = myFirstItem.ItemElement.Position curItem.Parent = myFirstItem.ItemElement.Parent end
		end
		
		if myFirstItem and myFirstItem.ItemElement then
			SlotData.ItemManager = myFirstItem.ItemManager
			
			myFirstItem.ItemElement.TextFrame.Size = myFirstItem.PotentialRotation % 2 == 1 and UDim2.fromOffset(myFirstItem.ItemElement.Size.Y.Offset, myFirstItem.ItemElement.Size.X.Offset) or UDim2.fromOffset(myFirstItem.ItemElement.Size.X.Offset, myFirstItem.ItemElement.Size.Y.Offset)
			myFirstItem.ItemElement.TextFrame.Rotation = myFirstItem.PotentialRotation * -90
			
			myFirstItem.ItemElement.TextFrame.ItemName.Size = UDim2.fromScale(0.95, myFirstItem.PotentialRotation % 2 == 1 and 0.1 or 0.3)
			myFirstItem.ItemElement.TextFrame.Count.Size = UDim2.fromScale(0.95, myFirstItem.PotentialRotation % 2 == 1 and 0.1 or 0.3)
			
			if inInventory then				
				if tool.Value ~= "Bullets" then
					myFirstItem.ItemElement.TextFrame.Count.Text = (tool:GetAttribute("MaxHealth") > 5) and `{math.floor(tool:GetAttribute("Health"))}/{ArmorDB[tool.Value][`TIER_{tool.Tier.Value}`].durability}` or ""
					myFirstItem.ItemElement.TextFrame.Count.TextColor3 = (tool:GetAttribute("Health") <= 0) and Color3.fromRGB(147, 6, 2):Lerp(Color3.fromRGB(200, 65, 67), math.abs(math.sin(tick() * 2.5) * 0.5)) or Color3.fromRGB(255,255,255)
				else
					myFirstItem.ItemElement.TextFrame.Count.Text = `{tool:GetAttribute("ammo")}`
				end
			else
				if not tool:GetAttribute("HealItem") then
					if not tool:GetAttribute("Magazine") then
						local ammocolor1 = (tool:FindFirstChild("_data") and (tool._data:FindFirstChild("ammo") and tool._data:FindFirstChild("magCurrent"))) and Color3.fromRGB(170, 0, 0):Lerp(Color3.fromRGB(255, 255, 255), (tool._data.magCurrent.Value / tool._data.magSize.Value) * 1):ToHex() or Color3.fromRGB(0,0,0)
						local ammocolor2 = (tool:FindFirstChild("_data") and (tool._data:FindFirstChild("ammo") and tool._data:FindFirstChild("magCurrent"))) and tool._data.ammo.Value > 0 and Color3.fromRGB(255, 255, 255):ToHex() or Color3.fromRGB(79, 0, 0):ToHex() or Color3.fromRGB(0,0,0)

						myFirstItem.ItemElement.TextFrame.Count.Text = (tool:FindFirstChild("_data") and (tool._data:FindFirstChild("ammo") and tool._data:FindFirstChild("magCurrent"))) and `<font color="#{ammocolor1}">{tool._data.magCurrent.Value}</font>/<font color="#{ammocolor2}">{tool._data.ammo.Value}</font>` or ""
					else
						myFirstItem.ItemElement.TextFrame.Count.Text = (tool:FindFirstChild("_data") and (tool._data:FindFirstChild("max") and tool._data:FindFirstChild("current"))) and `{tool._data.current.Value}/{tool._data.max.Value}` or ""
					end
				else
					local healmodule = require(RP.Storage.Modules.Foods:FindFirstChild(tool.Name))
					
					local usescolor = (tool:GetAttribute("healpoints") and healmodule.healpoints) and Color3.fromRGB(170, 0, 0):Lerp(Color3.fromRGB(255, 255, 255), (tool:GetAttribute("healpoints") / healmodule.healpoints) * 1) or Color3.fromRGB(0,0,0)
					myFirstItem.ItemElement.TextFrame.Count.Text = (tool:GetAttribute("healpoints") and healmodule.healpoints) and `{math.floor(tool:GetAttribute("healpoints"))}/{healmodule.healpoints}` or ""
					myFirstItem.ItemElement.TextFrame.Count.TextColor3 = usescolor
				end
			end
		end
	end)

	return SlotData
end

function __class:CreateLootGrid(range:{number}, name:string, lootname:string)
	local Trove: TroveModule.TroveType = TroveModule.new()

	self.statsui.Stats.Loot.ScrollingFrame.LootObject.TextLabel.Text = ` {lootname}`
	
	self.lootgrid = GridPack.createGrid({
		Parent = self.statsui.Stats.Loot.ScrollingFrame.LootObject.Slots,

		Visible = true,

		Assets = {
			Slot = script.slotAsset,
		},

		GridSize = Vector2.new(range[1], range[2]), -- How many slots the grid has on the X and Y axes.
		
		UseFrame = true,
		
		GridCellSize = UDim2.fromOffset(42,42),
		MaxCells = 8,

		AnchorPoint = Vector2.new(0, 0), -- Anchor point of the grid container.
		Position = UDim2.new(0, 0, 0, 0), -- Position of the grid container.
		Size = UDim2.fromScale(1,1), -- Size of the grid container.

		Metadata = {
			Type = "Loot",
			LootName = name,
			FakeName = lootname,
		},
	})
	self.statsui.Stats.Loot.ScrollingFrame.LootObject.TextLabel.Size = UDim2.fromScale(1.4*(self.lootgrid.GridSize.X/8.325), 0.086)
	
	self.lootgrid:ConnectTransferLink(self.transferLink)
	
	self.lootgrid.GuiElement.Destroying:Connect(function()
		Trove:Destroy()
	end)
	
	Trove:Connect(RS.RenderStepped, function(dt)		
		local dt2 = dt*60
		self.statsui.Stats.Loot.ScrollingFrame.LootObject.TextLabel.LoadingFrame.Loading.Rotation += 2*dt2
		
		local isloading = false
		for _,v in pairs(self.lootgrid.GuiElement.Parent:GetChildren()) do
			if v.Name == "InvItem" and not v.ViewportFrame.Visible then
				isloading = true
			end
		end
		self.statsui.Stats.Loot.ScrollingFrame.LootObject.TextLabel.LoadingFrame.Visible = isloading
	end)
end

local exist_lootfunc = false

function __class:AddLootItem(toolname:string, uniqueid:string, toolmodel:any, invsize:string, isArmor:boolean, posInfo:{any}, toolInfo:{any})
	local Trove: TroveModule.TroveType = TroveModule.new()
	local invitem_temp = script.InvItem:Clone()
	
	invitem_temp.TextFrame.ItemName.Text = (isArmor or toolInfo.ismiscitem) and toolmodel:GetAttribute("gridName") or toolname
	
	local loading = false
	
	local lootItem = GridPack.createItem({
		Position = posInfo and Vector2.new(posInfo.X, posInfo.Y) or Vector2.new(0, 0),
		Size = WeaponSizes[invsize] or Vector2.new(2, 3),
		
		Rotation = posInfo and posInfo.Rot or 0,

		Assets = {
			Item = invitem_temp,
		},

		MoveMiddleware = function(movedItem, newGridPosition, newRotation, lastItemManager, newItemManager)
        --[[
            movedItem: This Item
            newGridPosition: This Item's new position in a Grid. (Doesn't apply with SingleSlots)
            newRotation: This Item's new rotation.
            lastItemManager: The ItemManager that the Item was in before it got moved.
            newItemManager: The new ItemManager the item was moved to. (If there is one)
        ]]
			movedItem.ItemElement.UIStroke.Thickness = 1
			movedItem.ItemElement.InteractionButton.BackgroundTransparency = 0.15
		
			if not newItemManager then return end
			
			if newItemManager.Metadata.MainType == "Armor" then
				if newItemManager.Metadata.Type ~= toolInfo.armortype then return false end

				local armorfound = newItemManager:IsColliding(movedItem, {})
				if armorfound then return false end
			end
			
			if newItemManager.Metadata.MainType == "Weapons" then
				if newItemManager.Metadata.Type == "Primary" and (not toolInfo.gun or toolInfo.secondary) then return false end
				if newItemManager.Metadata.Type == "Secondary" and not toolInfo.secondary then return false end
				if newItemManager.Metadata.Type == "Melee" and not toolInfo.melee then return false end
			end
			
			if newItemManager.Metadata.MainType == "Limbs" and not toolInfo.ishealitem then return false end
			
			if newItemManager.Metadata.MainType == "Hotbar" and (toolInfo.melee or toolInfo.gun or toolInfo.armortype) then return false end
			
			if newItemManager.Metadata.Type ~= "Loot" then
				loading = true
				-- invoke "place tool into backpack" function onto server
				movedItem.ItemElement.Loading.Visible = true
				movedItem.ItemElement.InteractionButton.Visible = false
				movedItem.ItemElement.ViewportFrame.BackgroundTransparency = 0
				movedItem.ItemElement.Fade.BackgroundColor3 = Color3.fromRGB(0,0,0)
				movedItem.ItemElement.Fade.BackgroundTransparency = 0.5
				
				game:GetService("RunService"):BindToRenderStep(`loading_{uniqueid}`, Enum.RenderPriority.Camera.Value, function(dt)
					if not movedItem.ItemElement then
						game:GetService("RunService"):UnbindFromRenderStep(`loading_{uniqueid}`)
						return
					end
					
					local dt2 = dt*60
					movedItem.ItemElement.Loading.Rotation += 2*dt2
				end)
				
				local inwepslot = nil
				local inarmorslot = nil
				if newItemManager.Metadata.MainType == "Weapons" and (toolInfo.gun or toolInfo.melee) then inwepslot = newItemManager.Metadata.Index end
				if newItemManager.Metadata.MainType == "Armor" and isArmor and newItemManager.Metadata.Type ~= "FaceArmor" then inarmorslot = true end
				
				task.delay(0.15, function()
					local itemexists = Network:InvokeServer("varsFunction", "lootConfirm", {
						lootName = lastItemManager.Metadata.LootName,
						ID = uniqueid,
					})
					if itemexists then
						Network:FireServer("vars", "lootServer", {lastItemManager.Metadata.LootName, uniqueid, toolmodel, {X = newGridPosition.X, Y = newGridPosition.Y, Rot = movedItem.Rotation, Type = newItemManager.Metadata.Type == "FaceArmor" and "Modify" or newItemManager.Metadata.Type, WepSlot = inwepslot, ArmorSlot = inarmorslot, HotbarSlot = newItemManager.Metadata.MainType == "Hotbar" and newItemManager.Metadata.Index-4 or nil}})
						task.wait()
						table.remove(self.alreadyLooted, table.find(self.alreadyLooted, uniqueid))
						newItemManager:RemoveItem(movedItem)
					else
						table.remove(self.alreadyLooted, table.find(self.alreadyLooted, uniqueid))
						newItemManager:RemoveItem(movedItem)
					end
				end)
				return
			end

			-- If the result if false then the Item will move back to it's last position.
		end,
		
		RenderMiddleware = function(draggingItem)
			draggingItem.ItemElement.UIStroke.Thickness = 3
			draggingItem.ItemElement.InteractionButton.BackgroundTransparency = 0.5
		end,
		
		ClickMiddleware = function(clickedItem, mouseLocation)
			if not self.ui.Parent.ActionBox.Visible then self.ui.Parent.ActionBox.Visible = true end

			self.currentAction = clickedItem

			local frameSize = self.ui.Parent.ActionBox.AbsoluteSize
			self.ui.Parent.ActionBox.Position = UDim2.fromOffset(mouseLocation.X - frameSize.X, mouseLocation.Y + (frameSize.Y*1.75))
		end,

		Metadata = {},
	})
	self.lootgrid:AddItem(lootItem, nil, not posInfo)
	
	if not posInfo then
		task.spawn(function()
			if table.find(self.alreadyLooted, uniqueid) then 
				for i, v in Rarity do
					for i1, v1 in v.Items do
						if v1 == toolname then
							lootItem.ItemElement.InteractionButton.BackgroundColor3 = v.Color
							lootItem.ItemElement.ViewportFrame.BackgroundColor3 = v.Color
						end
					end
				end 
				return 
			end
			
			lootItem.ItemElement.InteractionButton.Visible = false
			lootItem.ItemElement.ViewportFrame.Visible = false
			lootItem.ItemElement.UIStroke.Enabled = false
			lootItem.ItemElement.BackgroundTransparency = 0
			lootItem.ItemElement.TextFrame.Visible = false

			local random_num = Random.new():NextNumber(0.75,1.25)

			local lootsearch = task.delay(lootItem.Size.Magnitude*random_num, function()
				table.insert(self.alreadyLooted, uniqueid)

				lootItem.ItemElement.InteractionButton.Visible = true
				lootItem.ItemElement.ViewportFrame.Visible = true
				lootItem.ItemElement.TextFrame.Visible = true
				lootItem.ItemElement.UIStroke.Enabled = true
				lootItem.ItemElement.BackgroundTransparency = 1

				for i, v in Rarity do
					for i1, v1 in v.Items do
						if v1 == toolname then
							lootItem.ItemElement.InteractionButton.BackgroundColor3 = v.Color
							lootItem.ItemElement.ViewportFrame.BackgroundColor3 = v.Color
						end
					end
				end

				lootItem.ItemElement.Fade.BackgroundTransparency = 0
				TS:Create(lootItem.ItemElement.Fade, TweenInfo.new(0.15), {BackgroundTransparency = 1}):Play()
			end)

			lootItem.ItemElement.Destroying:Once(function()
				if not table.find(self.alreadyLooted, uniqueid) then
					task.cancel(lootsearch)
				end
			end)
		end)
	else
		table.insert(self.alreadyLooted, uniqueid)
		
		for i, v in Rarity do
			for i1, v1 in v.Items do
				if v1 == toolname then
					lootItem.ItemElement.InteractionButton.BackgroundColor3 = v.Color
					lootItem.ItemElement.ViewportFrame.BackgroundColor3 = v.Color
				end
			end
		end
	end
	
	local viewsettings = WeaponView[toolname]

	local cam = Instance.new("Camera")
	cam.FieldOfView = typeof(viewsettings) == "table" and viewsettings[2] or 70
	cam.Parent = lootItem.ItemElement.ViewportFrame

	lootItem.ItemElement.ViewportFrame.CurrentCamera = cam

	local mod = Instance.new("Model", lootItem.ItemElement.ViewportFrame)

	local model = toolmodel
	model = model:Clone()
	model.Parent = mod
	
		--[[local cf = cam.CFrame+(cam.CFrame.LookVector*1)
	cf *= WeaponView[tool.Name] or CFrame.identity
	
	mod:PivotTo(cf)]]

	local vpfModel = ViewportModel.new(lootItem.ItemElement.ViewportFrame, cam)
	local cf, size = mod:GetBoundingBox()

	vpfModel:SetModel(mod)

	--[[local distance = vpfModel:GetFitDistance(cf.Position)
	cam.CFrame = CFrame.new(cf.Position) * CFrame.new(0, 0, distance)]]
	local orientation = CFrame.fromEulerAnglesYXZ(0, 0, 0)
	cam.CFrame = (typeof(viewsettings) == "table" and viewsettings[1] or viewsettings) or vpfModel:GetMinimumFitCFrame(orientation)
	
	lootItem.ItemElement.Destroying:Once(function()
		if not loading then Network:FireServer("vars", "lootServer", {"", nil, toolmodel}) end
		Trove:Destroy()
	end)
	
	Trove:Connect(lootItem.ItemElement.InteractionButton.MouseButton1Down, function()
		if not self.enabled or self.currentAction ~= lootItem then return end
		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)

	Trove:Connect(self.ui.Parent.ActionBox.Inspect.MouseButton1Click, function()
		if not self.enabled or self.currentAction ~= lootItem then return end
		if self.ui.Parent:FindFirstChild(`hover_{uniqueid}`) then return end
		--print(`inspect {toolname} with id {uniqueid} pls`)

		local hoverinfo = self.player.PlayerGui.UI.HUD_GAMEPLAY.HoverInfo:Clone()
		hoverinfo.Parent = self.player.PlayerGui.UI.HUD_GAMEPLAY
		hoverinfo.Name = `hover_{uniqueid}`
		hoverinfo.Visible = true

		local viewportModelRotator = require(game:GetService('ReplicatedStorage').ViewportRotation)

		hoverinfo.Frame.ItemName.Frame.SettingsClose.MouseButton1Click:Connect(function()
			viewportModelRotator:Disable(hoverinfo.Frame.Frame.ViewportFrame)
			game:GetService("Debris"):AddItem(hoverinfo, 1/99)
			--task.wait()
			--hoverinfo:Destroy()
		end)		
		self.statsui:GetPropertyChangedSignal("Visible"):Once(function()
			print(hoverinfo:GetChildren())
			viewportModelRotator:Disable(hoverinfo.Frame.Frame.ViewportFrame)
			game:GetService("Debris"):AddItem(hoverinfo, 1)
			--task.wait()
			--hoverinfo:Destroy()
		end)

		local config = (RP.Storage.Modules.Weapons:FindFirstChild(toolname) and require(RP.Storage.Modules.Weapons:FindFirstChild(toolname))) or (RP.Storage.Modules.Magazines:FindFirstChild(toolname) and require(RP.Storage.Modules.Magazines:FindFirstChild(toolname))) or nil
		local mouse:Mouse = self.player:GetMouse()

		local frameSize = hoverinfo.AbsoluteSize
		hoverinfo.Position = UDim2.fromOffset(mouse.X+(frameSize.X*0.5), mouse.Y+(frameSize.Y*0.65))

		hoverinfo.Frame.Frame.ItemInfo.TextLabel.Text = (isArmor and (toolInfo.armorinfo or "")) or (toolInfo.ismiscitem and (toolInfo.caliberinfo or "")) or (config.description or "")
		hoverinfo.Frame.ItemName.Frame.ItemText.Text = toolInfo.armorname or toolname

		local templateframe = hoverinfo.Frame.Frame.StatsInfo.Main.template:FindFirstChild("templateFrame")

		local names = {
			["rate"] = "fire rate",
			["mode"] = "types of fire",
			["vel"] = "muzzle velocity",
		}
		local firemodes = {
			"single fire",
			"burst fire",
			"full auto",
		}
		local firemodes_min = {
			"single",
			"burst",
			"auto",
		}

		if config then
			for i,v in config do
				if toolInfo.gun then
					local name = tostring(i) 
					if name == "damage" or name == "rate" or name == "mode" or name == "vel" or name == "recoil" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						if typeof(v) == "table" and name == "damage" then					
							frame.StatName.Text = string.upper(name)
							frame.StatValue.Text = v[1] == v[2] and `{v[1]}` or `{v[1]}~{v[2]}`
							frame.Bar.Size = UDim2.fromScale(math.clamp(v[2]/100, 0, 1), 0.9)
						elseif typeof(v) == "table" and name == "recoil" then
							frame.StatName.Text = string.upper("horizontal recoil")
							frame.StatValue.Text = math.abs(v.AngleY_Min) == v.AngleY_Max and `{math.floor(v.AngleY_Max)}` or `{math.abs(math.floor(v.AngleY_Min))}~{math.floor(v.AngleY_Max)}`
							frame.Bar.Size = UDim2.fromScale(math.clamp(v.AngleY_Max/5, 0, 1), 0.9)

							local frame2 = templateframe:Clone()
							frame2.Name = `{name}2`
							frame2.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

							frame2.StatName.Text = string.upper("vertical recoil")
							frame2.StatValue.Text = math.abs(v.AngleX_Min) == v.AngleX_Max and `{math.floor(v.AngleX_Max)}` or `{math.abs(math.floor(v.AngleX_Min))}~{math.floor(v.AngleX_Max)}`
							frame2.Bar.Size = UDim2.fromScale(math.clamp(v.AngleX_Max/30, 0, 1), 0.9)

							frame2.Visible = true
						elseif typeof(v) == "number" then
							local hasswitch = name == "mode" and config.mode_switch
							local available_modes = hasswitch and (config.mode_switch[3] and `{firemodes_min[config.mode_switch[1]]}/{firemodes_min[config.mode_switch[2]]}/{firemodes_min[config.mode_switch[3]]}` or `{firemodes_min[config.mode_switch[1]]}/{firemodes_min[config.mode_switch[2]]}`) or nil

							frame.StatName.Text = string.upper(names[name])
							frame.StatValue.Text = (name == "mode" and (hasswitch and string.upper(available_modes) or `{string.upper(firemodes[v])}`)) or (name == "rate" and `~{v}s`) or `~{v} m/s`
							frame.Bar.Visible = false
						end

						frame.Visible = true
					end
				elseif toolInfo.melee then
					local name = tostring(i) 
					if name == "damage" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper(name)
						frame.StatValue.Text = v[1] == v[2] and `{v[1]}` or `{v[1]}~{v[2]}`
						frame.Bar.Size = UDim2.fromScale(math.clamp(v[2]/100, 0, 1), 0.9)

						frame.Visible = true
					end
				elseif toolInfo.ishealitem then
					local name = tostring(i) 
					if name == "maxuses" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper("heal unit")
						frame.StatValue.Text = v
						frame.Bar.Visible = false

						frame.Visible = true
					end
				elseif toolInfo.magazine then
					local name = tostring(i) 
					if name == "max" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper("magazine capacity")
						frame.StatValue.Text = v
						frame.Bar.Visible = false

						frame.Visible = true
					elseif name == "caliber_name" then
						local frame = templateframe:Clone()
						frame.Name = name
						frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

						frame.StatName.Text = string.upper("caliber")
						frame.StatValue.Text = v
						frame.Bar.Visible = false

						frame.Visible = true
					end
				end
			end
		end

		if isArmor then
			if toolInfo.canslide ~= nil then
				local frame = templateframe:Clone()
				frame.Name = "canslide"
				frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

				frame.StatName.Text = string.upper("can slide")
				frame.StatValue.Text = toolInfo.canslide and "YES" or "NO"
				frame.Bar.Visible = false

				frame.Visible = true
			end
			if toolInfo.sizeX ~= nil and toolInfo.sizeY ~= nil then
				local frame = templateframe:Clone()
				frame.Name = "capacity"
				frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

				frame.StatName.Text = string.upper("capacity")
				frame.StatValue.Text = `{toolInfo.sizeX*toolInfo.sizeY}`
				frame.Bar.Size = UDim2.fromScale(math.clamp((toolInfo.sizeX*toolInfo.sizeY)/100, 0, 1), 0.9)

				frame.Visible = true
			end
			if toolInfo.maxhealth > 5 then
				local frame = templateframe:Clone()
				frame.Name = "durability"
				frame.Parent = hoverinfo.Frame.Frame.StatsInfo.Main

				frame.StatName.Text = string.upper("durability")
				frame.StatValue.Text = `{toolInfo.maxhealth}`
				frame.Bar.Visible = false

				frame.Visible = true
			end
		end

		local mod = Instance.new("Model", hoverinfo.Frame.Frame.ViewportFrame)
		
		local model = toolmodel
		model = model:Clone()
		model.Parent = mod

		if model:GetAttribute("useY") then mod:SetAttribute("useY", true) end
		if model:GetAttribute("inverseY") then mod:SetAttribute("inverseY", true) end
		if model:GetAttribute("bothY") then mod:SetAttribute("bothY", true) end

		local cam = Instance.new("Camera")
		cam.Parent = hoverinfo.Frame.Frame.ViewportFrame

		hoverinfo.Frame.Frame.ViewportFrame.CurrentCamera = cam	
		cam.CFrame = (typeof(viewsettings) == "table" and viewsettings[1] or viewsettings) or vpfModel:GetMinimumFitCFrame(orientation)

		local pos = mod:GetBoundingBox()

		local part = Instance.new("Part", mod)
		part.Name = "RotPart"
		part.Size = Vector3.new(0.05,0.05,0.05)
		part.Transparency = 1
		part.CanCollide = false
		part.CFrame = pos

		mod.PrimaryPart = part

		viewportModelRotator:Enable(hoverinfo.Frame.Frame.ViewportFrame,3.5)

		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)
	Trove:Connect(self.ui.Parent.ActionBox.Discard.MouseButton1Click, function()
		if not self.enabled or self.currentAction ~= lootItem then return end

		local canDrop = Network:InvokeServer("varsFunction", "lootCantDrop", uniqueid)
		if not canDrop or isArmor or toolInfo.ismiscitem then
			_G.messageLog({
				Text = "YOU CAN'T DROP THIS ITEM",
				TextColor3 = Color3.fromRGB(255, 94, 97),
			}, false, 2)
		else
			Network:FireServer("pass", nil, self.player.Character.HumanoidRootPart.Velocity + Vector3.new(0,1,0), uniqueid)
			lootItem.ItemManager:RemoveItem(lootItem)
		end

		self.ui.Parent.ActionBox.Visible = false
		self.currentAction = nil
	end)
	
	Trove:Connect(lootItem.ItemElement.InteractionButton.MouseButton1Click, function()
		if not self.enabled or exist_lootfunc then return end

		if UIS:IsKeyDown(Enum.KeyCode.LeftControl) or UIS:IsKeyDown(Enum.KeyCode.RightControl) then
			local pocketsFull = false
			local backpackFull = false
			
			local nextFreePosition = self.grid:GetNextFreePositionForItem(lootItem)
			if not nextFreePosition then
				nextFreePosition = self.grid:GetNextFreePositionForItem(lootItem, lootItem.Rotation+1)
				if not nextFreePosition then
					pocketsFull = true
				end
			end
			
			if self.Backpack then
				local nextFreePosition = self.Backpack:GetNextFreePositionForItem(lootItem)
				if not nextFreePosition then
					nextFreePosition = self.Backpack:GetNextFreePositionForItem(lootItem, lootItem.Rotation+1)
					if not nextFreePosition then
						backpackFull = true
					end
				end
			else
				backpackFull = true
			end
			
			if pocketsFull and backpackFull then return end
			
			loading = true
			-- invoke "place tool into backpack" function onto server
			lootItem.ItemElement.Loading.Visible = true
			lootItem.ItemElement.InteractionButton.Visible = false
			lootItem.ItemElement.ViewportFrame.BackgroundTransparency = 0
			lootItem.ItemElement.Fade.BackgroundColor3 = Color3.fromRGB(0,0,0)
			lootItem.ItemElement.Fade.BackgroundTransparency = 0.5

			game:GetService("RunService"):BindToRenderStep(`loading_{uniqueid}`, Enum.RenderPriority.Camera.Value, function(dt)
				if not lootItem.ItemElement then
					game:GetService("RunService"):UnbindFromRenderStep(`loading_{uniqueid}`)
					return
				end

				local dt2 = dt*60
				lootItem.ItemElement.Loading.Rotation += 2*dt2
			end)
			
			exist_lootfunc = true

			task.delay(0.15, function()
				local itemexists = Network:InvokeServer("varsFunction", "lootConfirm", {
					lootName = lootItem.ItemManager.Metadata.LootName,
					ID = uniqueid,
				})
				if itemexists then
					Network:FireServer("vars", "lootServer", {lootItem.ItemManager.Metadata.LootName, uniqueid, toolmodel, {Type = pocketsFull and self.Backpack.Metadata.Type or self.grid.Metadata.Type, Rot = nil}})
					task.wait()
					table.remove(self.alreadyLooted, table.find(self.alreadyLooted, uniqueid))
					lootItem.ItemManager:RemoveItem(lootItem)
				else
					table.remove(self.alreadyLooted, table.find(self.alreadyLooted, uniqueid))
					lootItem.ItemManager:RemoveItem(lootItem)
				end
				
				exist_lootfunc = false
			end)
		elseif UIS:IsKeyDown(Enum.KeyCode.LeftAlt) or UIS:IsKeyDown(Enum.KeyCode.RightAlt) then
			if not self.enabled or exist_lootfunc then return end
			
			local slotFull = false
			local wepSlot = nil
			local hotbarSlot = nil
			
			if isArmor then
				local armorfound = self[`{string.lower(toolInfo.armortype)}Slot`]:IsColliding(lootItem, {})
				if armorfound then slotFull = true end
			end

			if (toolInfo.gun or toolInfo.melee) then
				local weapon1found = self.wepSlot1:IsColliding(lootItem, {})
				local weapon2found = self.wepSlot2:IsColliding(lootItem, {})
				local weapon3found = self.wepSlot3:IsColliding(lootItem, {})
				local weapon4found = self.wepSlot4:IsColliding(lootItem, {})
				
				if (toolInfo.gun and not toolInfo.secondary) and weapon1found then 
					if weapon2found then slotFull = true else wepSlot = 2 end
				elseif (toolInfo.gun and not toolInfo.secondary) and not weapon1found then
					wepSlot = 1
				end
				if toolInfo.secondary and weapon3found then slotFull = true elseif toolInfo.secondary and not weapon3found then wepSlot = 3 end
				if toolInfo.melee and weapon4found then slotFull = true elseif toolInfo.melee and not weapon4found then wepSlot = 4 end
			end
			
			if not isArmor and not toolInfo.gun and not toolInfo.melee and not toolInfo.ismiscitem then
				local foundslots = {
					self.hotbarSlot1:IsColliding(lootItem, {}),
					self.hotbarSlot2:IsColliding(lootItem, {}),
					self.hotbarSlot3:IsColliding(lootItem, {}),
					self.hotbarSlot4:IsColliding(lootItem, {}),
					self.hotbarSlot5:IsColliding(lootItem, {}),
				}
				
				for i,v in pairs(foundslots) do
					if not v then hotbarSlot = i break end
				end
				if hotbarSlot == nil then slotFull = true end
			end

			if slotFull then return end

			loading = true
			-- invoke "place tool into backpack" function onto server
			lootItem.ItemElement.Loading.Visible = true
			lootItem.ItemElement.InteractionButton.Visible = false
			lootItem.ItemElement.ViewportFrame.BackgroundTransparency = 0
			lootItem.ItemElement.Fade.BackgroundColor3 = Color3.fromRGB(0,0,0)
			lootItem.ItemElement.Fade.BackgroundTransparency = 0.5

			game:GetService("RunService"):BindToRenderStep(`loading_{uniqueid}`, Enum.RenderPriority.Camera.Value, function(dt)
				if not lootItem.ItemElement then
					game:GetService("RunService"):UnbindFromRenderStep(`loading_{uniqueid}`)
					return
				end

				local dt2 = dt*60
				lootItem.ItemElement.Loading.Rotation += 2*dt2
			end)
			
			exist_lootfunc = true

			task.delay(0.15, function()
				local itemexists = Network:InvokeServer("varsFunction", "lootConfirm", {
					lootName = lootItem.ItemManager.Metadata.LootName,
					ID = uniqueid,
				})
				if itemexists then
					Network:FireServer("vars", "lootServer", {lootItem.ItemManager.Metadata.LootName, uniqueid, toolmodel, {Type = (toolInfo.armortype == "FaceShield" or toolInfo.armortype == "NVG") and "FaceArmor" or nil,WepSlot = wepSlot, ArmorSlot = (toolInfo.armortype ~= "FaceShield" and toolInfo.armortype ~= "NVG") and toolInfo.armortype or nil, HotbarSlot = hotbarSlot}})
					task.wait()
					table.remove(self.alreadyLooted, table.find(self.alreadyLooted, uniqueid))
					lootItem.ItemManager:RemoveItem(lootItem)
				else
					table.remove(self.alreadyLooted, table.find(self.alreadyLooted, uniqueid))
					lootItem.ItemManager:RemoveItem(lootItem)
				end
				
				exist_lootfunc = false
			end)
		end
	end)
	
	Trove:Connect(RS.RenderStepped, function()
		if lootItem then
			lootItem.ItemElement.TextFrame.Size = lootItem.Rotation % 2 == 1 and UDim2.fromOffset(lootItem.ItemElement.Size.Y.Offset, lootItem.ItemElement.Size.X.Offset) or UDim2.fromOffset(lootItem.ItemElement.Size.X.Offset, lootItem.ItemElement.Size.Y.Offset)
			lootItem.ItemElement.TextFrame.Rotation = lootItem.Rotation * -90

			lootItem.ItemElement.TextFrame.ItemName.Size = UDim2.fromScale(0.95, lootItem.Rotation % 2 == 1 and 0.1 or 0.3)
			lootItem.ItemElement.TextFrame.Count.Size = UDim2.fromScale(0.95, lootItem.Rotation % 2 == 1 and 0.1 or 0.3)

			if isArmor then
				lootItem.ItemElement.TextFrame.Count.Text = ""
			else
				if not toolInfo.ishealitem then
					if toolInfo.magazine then
						lootItem.ItemElement.TextFrame.Count.Text = (toolInfo.info.current and toolInfo.info.max) and `{toolInfo.info.current}/{toolInfo.info.max}` or ""
					elseif toolInfo.ismiscitem then
						lootItem.ItemElement.TextFrame.Count.Text = toolInfo.info.bullets and `{toolInfo.info.bullets}` or ""
					else
						lootItem.ItemElement.TextFrame.Count.Text = (toolInfo.info.magCurrent and toolInfo.info.ammo) and `{toolInfo.info.magCurrent}/{toolInfo.info.ammo}` or ""
					end
				else
					lootItem.ItemElement.TextFrame.Count.Text = (toolInfo.info.uses and toolInfo.info.max) and `{toolInfo.info.uses}/{toolInfo.info.max}` or ""
				end
			end
		end
	end)
	
	table.insert(self.lootItems, lootItem)
end
function __class:DestroyLootGrid(refresh)
	if self.lootgrid then
	
		self.lootgrid:DisconnectTransferLink(self.transferLink)
		
		local range = {self.lootgrid.GridSize.X, self.lootgrid.GridSize.Y}
		local name = self.lootgrid.Metadata.LootName
		local lootname = self.lootgrid.Metadata.FakeName
		
		self.lootgrid:Destroy()			
		self.lootItems = {}

		for _,v in self.statsui.Stats.Loot.ScrollingFrame.LootObject.Slots:GetChildren() do
			if v:IsA("CanvasGroup") or v:IsA("Frame") then v:Destroy() end
		end
		
		if refresh then		
			self:CreateLootGrid(range, name, lootname)			
			task.wait()
			Network:FireServer("vars", "lootRefresh", name)
		end
	end
end

function __class:storeBackpackItems(Tool:Tool)
	if Tool.Value == "Backpack" and Tool:GetAttribute("wearing") then
		for i,v in self.storedItems do
			Network:FireServer("vars", "storeBackpackItems", {Tool, i, v})
			self.Backpack:RemoveItem(self.slots[i].Item)
		end
		self.storedItems = {}
		Network:FireServer("vars", "wearBackpack", {Tool, nil})
	end
end

function __class:DestroySlot(Tool: Tool)
	local SlotData: SlotData? = self.slots[Tool]
    
	if not SlotData then return end
	if not SlotData.hasInventory then		
		if self.ui.Hotbar[tostring(SlotData.Index)]:FindFirstChild("ItemIcon") then
			local gunicon = self.ui.Hotbar[tostring(SlotData.Index)]:FindFirstChild("ItemIcon")
			if gunicon then
				gunicon:Destroy()
			end
			self.ui.Hotbar[tostring(SlotData.Index)].SelectedItem.Visible = false
		end
		if SlotData.SlotFrame ~= nil then
			SlotData.SlotFrame:Destroy()
			self.ui.Hotbar[tostring(SlotData.Index)].SelectedItem.Visible = false
			--[[if SlotData.ItemManager.GuiElement.Parent.Icon ~= nil then
				SlotData.ItemManager.GuiElement.Parent.Icon.Visible = true
			end]]
			
		end
		
		--[[if SlotData.Frame.Name ~= "GunImage" then
			if SlotData.Frame.Parent:FindFirstChild("SelectedItem") then
				SlotData.Frame.Parent.SelectedItem.Visible = false
			end
		else
			local gunicon = self.ui.Hotbar[tostring(SlotData.Index)]:FindFirstChild("ItemIcon")
			if gunicon then
				gunicon:Destroy()
			end
			self.ui.Hotbar[tostring(SlotData.Index)].SelectedItem.Visible = false
		
		end]]

		game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("tool", false)
	else
		game:GetService('ReplicatedStorage').Storage.Events.changeCapacity:FireServer("inv", false)
	end
	
	if SlotData.SlotFrame then SlotData.SlotFrame:Destroy() end
	if SlotData.ItemManager and SlotData.Item then SlotData.ItemManager:RemoveItem(SlotData.Item) end
	SlotData.Trove:Destroy()

	self.slots[Tool] = nil
end

function __class:Toggle(Enabled: boolean)
	self.enabled = Enabled
end

return inv

