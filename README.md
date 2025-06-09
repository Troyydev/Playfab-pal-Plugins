# PlayFabPal SDK

> A lightweight, pluggable SDK for building Discord slash‐command plugins that call PlayFab admin/server APIs.

---

## Table of Contents

1. [Introduction](#introduction)  
2. [Installation](#installation)  
3. [Quick Start](#quick-start)  
4. [Core Concepts](#core-concepts)  
   - [PluginAPI](#pluginapi)  
   - [PluginPayload](#pluginpayload)  
   - [SafePlayFab](#safeplayfab)  
   - [Decorators](#decorators)  
5. [Full API Reference](#full-api-reference)  
   1. [PluginAPI Methods](#pluginapi-methods)  
   2. [PluginPayload Methods](#pluginpayload-methods)  
   3. [SafePlayFab Methods](#safeplayfab-methods)  
6. [Example Plugin](#example-plugin)  
7. [Advanced: Modals & UI](#advanced-modals--ui)  
8. [FAQ](#faq)  
9. [License](#license)  

---

## Introduction

The **PlayFabPal SDK** makes it easy to write Discord slash‐command plugins that interact with PlayFab’s Admin & Server APIs.  
It handles:

- **Slash command registration** per-guild  
- **Payload construction & permission checks** via `SafePlayFab`  
- **HTTP call injection** (so your bot can configure keys securely)  
- **Modal dialogs** for richer input  
- A simple **decorator** interface  

---

## Installation

Copy the `playfabpal_sdk/` folder into your bot project.  
Ensure your directory structure:

