# Unreal Engine Cpp Development

[TOC]

## Class Hierarchy

* Object
* Actor
* Pawn
* Character
* MyCharacter

## Class Prefixes

* U - Classes Deriving from Uobject
* A - deriving from Actor
* F - Structs
* E -Enums
* I - Interfaces

### Examples: 

* Acharacter(Class)

* FHitResult(Struct)

* EDirection(Enum)

## Unreal Property System

### Unreal Macros

UPROPERTY, UFUNCTION, UCLASS, USTRUCT, UENUM 

### Marking up C++

* Editor & Blueprints Access
* Network behavior
* Memory Management

### Handles Boilerplate for us

## PlayerController

### Controller.h

Shared base for AI and players,not visually represented in world.

* Has 'ControlRotation' (for camera use etc.), separate from Pawn's Rotation

  **ControlRotation可以通过SpringArmComp传递给摄影机**

  ![image-20220621224143910](assets/image-20220621224143910.png)

  **ControlRotation可以传递给Pawn**

  ![image-20220621222829351](assets/image-20220621222829351.png)

  

* PlayerController.h

  One per player, persists for entire session (or until level changes)

  'possess' Pawn and forwards input from player

  Pawn can 'die' and destroyed while PlayerController persists

* AIController.h

  One per AI, holds the 'logic' that controls the Pawn
  
  ## Collision & Physics
  
  * Collision Channels
  * Object Types
  * Reactions: Block, Overlap, Ignore