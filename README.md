# Saba-101
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Floor Plan Schematic (30m x 20m)</title>
    <style>
        :root {
            --primary: #2563eb;
            --corridor: #f1f5f9;
            --room-bg: #ffffff;
            --wall-color: #334155;
            --text-color: #0f172a;
            --accent-wet: #e0f2fe;
            --accent-utility: #fef3c7;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        }

        body {
            background-color: #f8fafc;
            color: var(--text-color);
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        header {
            margin-bottom: 20px;
            text-align: center;
        }

        h1 {
            font-size: 1.5rem;
            color: #1e293b;
            margin-bottom: 5px;
        }

        p {
            font-size: 0.9rem;
            color: #64748b;
        }

        .controls {
            margin-bottom: 20px;
            background: white;
            padding: 10px 20px;
            border-radius: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            font-size: 0.85rem;
            display: flex;
            gap: 15px;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        .legend-color {
            width: 16px;
            height: 16px;
            border: 1px solid var(--wall-color);
            border-radius: 3px;
        }

        /* Floor Plan Container Scale: 1 meter = 30 pixels */
        /* Total Plan Viewport: 30m width * 30px = 900px, 20m height * 30px = 600px */
        .building-outer {
            position: relative;
            width: 900px;
            height: 600px;
            background-color: var(--room-bg);
            border: 6px solid var(--wall-color);
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
            overflow: hidden;
        }

        /* Windows Blueprint Visuals */
        .window-row-top, .window-row-bottom {
            position: absolute;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: space-around;
            pointer-events: none;
            z-index: 10;
        }

        .window-row-top { top: -6px; }
        .window-row-bottom { bottom: -6px; }

        .window-glass {
            width: 45px;
            height: 6px;
            background-color: #cbd5e1;
            border: 1px solid #0284c7;
            box-shadow: 0 0 4px #0ea5e9;
        }

        /* Layout Dividers (Main Structure) */
        .upper-zone {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 360px; /* 12 meters deep */
            display: flex;
        }

        .corridor-horizontal {
            position: absolute;
            top: 360px; /* Starts at 12m mark */
            left: 0;
            width: 100%;
            height: 60px; /* 2 meters wide corridor */
            background-color: var(--corridor);
            border-top: 3px solid var(--wall-color);
            border-bottom: 3px solid var(--wall-color);
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 600;
            font-size: 0.8rem;
            letter-spacing: 0.1em;
            color: #64748b;
            z-index: 2;
        }

        .lower-zone {
            position: absolute;
            top: 420px; /* Starts at 14m mark */
            left: 0;
            width: 100%;
            height: 120px; /* 4 meters deep small rooms */
            display: flex;
        }

        /* Grid Room Management Styling */
        .main-big-room {
            height: 100%;
            width: 33.333%;
            border-right: 3px solid var(--wall-color);
            position: relative;
            display: flex;
        }

        .main-big-room:last-child {
            border-right: none;
        }

        .sub-room-top {
            height: 100%;
            flex: 1;
            border-right: 1px dashed #94a3b8;
            position: relative;
            padding: 8px;
            font-size: 0.7rem;
            font-weight: 500;
            color: #475569;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }

        .sub-room-top:last-child {
            border-right: none;
        }

        .small-room-bottom {
            height: 100%;
            width: 99.9px; /* Starts at ~3.33m flexible footprint baseline */
            border-right: 2px solid var(--wall-color);
            position: relative;
            padding: 8px;
            font-size: 0.7rem;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            background-color: var(--room-bg);
            transition: background-color 0.2s;
        }

        .small-room-bottom:last-child {
            border-right: none;
        }

        /* Distinct Facility Spaces Styling */
        .wet-zone-shower {
            background-color: var(--accent-wet);
        }

        .utility-zone {
            background-color: var(--accent-utility);
        }

        /* Interactive Adjustable Dragger Sliders */
        .resize-handle-v {
            position: absolute;
            top: 0;
            right: -4px;
            width: 8px;
            height: 100%;
            cursor: col-resize;
            z-index: 5;
            transition: background 0.2s;
        }

        .resize-handle-v:hover, .resize-handle-v.active {
            background-color: var(--primary);
        }

        .room-label {
            font-weight: bold;
            color: #1e293b;
            margin-bottom: 2px;
        }

        .dimensions-badge {
            font-family: monospace;
            background: #f1f5f9;
            padding: 2px 4px;
            border-radius: 3px;
            display: inline-block;
            font-size: 0.65rem;
            color: #334155;
            width: fit-content;
        }
    </style>
</head>
<body>

    <header>
        <h1>Interactive Architectural Schematic</h1>
        <p>Scale Footprint Overall: 30m × 20m | Central Corridor Width: 2m | Draggable Room Configurations</p>
    </header>

    <div class="controls">
        <div class="legend-item">
            <div class="legend-color" style="background: var(--room-bg);"></div>
            <span>Standard Multi-Rooms</span>
        </div>
        <div class="legend-item">
            <div class="legend-color" style="background: var(--accent-wet);"></div>
            <span>Shower Facilities</span>
        </div>
        <div class="legend-item">
            <div class="legend-color" style="background: var(--accent-utility);"></div>
            <span>Utility / Toilet Core</span>
        </div>
        <div class="legend-item">
            <div class="legend-color" style="background: #cbd5e1; border: 1px solid #0284c7;"></div>
            <span>External Window Array (11 Top / 11 Bottom)</span>
        </div>
    </div>

    <!-- The Entire Building Matrix Frame -->
    <div class="building-outer" id="buildingCanvas">
        
        <!-- Top Wall Windows Array (11 Structural Openings) -->
        <div class="window-row-top">
            <div class="window-glass"></div><div class="window-glass"></div><div class="window-glass"></div>
            <div class="window-glass"></div><div class="window-glass"></div><div class="window-glass"></div>
            <div class="window-glass"></div><div class="window-glass"></div><div class="window-glass"></div>
            <div class="window-glass"></div><div class="window-glass"></div>
        </div>

        <!-- Upper Tier Spaces Layout (3 Master Blocks divided into 5 modules each) -->
        <div class="upper-zone">
            <!-- Block 1 -->
            <div class="main-big-room">
                <div class="sub-room-top"><span class="room-label">A-1</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">A-2</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">A-3</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">A-4</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">A-5</span><span class="dimensions-badge">2m×12m</span></div>
            </div>
            <!-- Block 2 -->
            <div class="main-big-room">
                <div class="sub-room-top"><span class="room-label">B-1</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">B-2</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">B-3</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">B-4</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">B-5</span><span class="dimensions-badge">2m×12m</span></div>
            </div>
            <!-- Block 3 -->
            <div class="main-big-room">
                <div class="sub-room-top"><span class="room-label">C-1</span><span class="dimensions-badge">2m×12m</span></div>
                <div class="sub-room-top"><span class="room-label">C-2</span><span class="dimensions-badge">2m×12m</span></div>
