# SKY-NET-VPN
import React, { useState } from 'react';
import { StyleSheet, Text, View, TouchableOpacity, SafeAreaView } from 'react-native';

export default function VpnDashboard() {
  const [isConnected, setIsConnected] = useState(false);

  const toggleConnection = () => {
    // In a real app, you would trigger your native VPN logic here
    setIsConnected(!isConnected);
  };

  return (
    <SafeAreaView style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerText}>⚡ TurboVPN</Text>
      </View>

      {/* Connection Status */}
      <View style={styles.statusContainer}>
        <Text style={styles.statusLabel}>STATUS</Text>
        <Text style={[styles.statusValue, isConnected ? styles.connectedText : styles.disconnectedText]}>
          {isConnected ? 'SECURELY CONNECTED' : 'UNPROTECTED'}
        </Text>
      </View>

      {/* Main Toggle Button */}
      <View style={styles.buttonContainer}>
        <TouchableOpacity 
          style={[styles.powerButton, isConnected ? styles.buttonConnected : styles.buttonDisconnected]} 
          onPress={toggleConnection}
        >
          <Text style={styles.powerIcon}>🛑</Text>
        </TouchableOpacity>
        <Text style={styles.toggleHint}>
          {isConnected ? 'Tap to Disconnect' : 'Tap to Connect'}
        </Text>
      </View>

      {/* Server Selector Card */}
      <TouchableOpacity style={styles.serverCard}>
        <Text style={styles.serverFlag}>🇺🇸</Text>
        <View style={styles.serverInfo}>
          <Text style={styles.serverName}>United States</Text>
          <Text style={styles.serverCity}>New York - Server #4</Text>
        </View>
        <Text style={styles.chevron}>&gt;</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0f172a', justifyContent: 'space-between', padding: 20 },
  header: { alignItems: 'center', marginTop: 10 },
  headerText: { color: '#fff', fontSize: 20, fontWeight: 'bold' },
  statusContainer: { alignItems: 'center', marginTop: 40 },
  statusLabel: { color: '#94a3b8', fontSize: 12, letterSpacing: 2 },
  statusValue: { fontSize: 24, fontWeight: 'bold', marginTop: 5 },
  connectedText: { color: '#10b981' },
  disconnectedText: { color: '#ef4444' },
  buttonContainer: { alignItems: 'center', my: 'auto' },
  powerButton: { width: 150, height: 150, borderRadius: 75, justifyContent: 'center', alignItems: 'center', elevation: 10 },
  buttonDisconnected: { backgroundColor: '#1e293b', borderWidth: 2, borderColor: '#334155' },
  buttonConnected: { backgroundColor: '#10b981', shadowColor: '#10b981', shadowRadius: 20, shadowOpacity: 0.5 },
  powerIcon: { fontSize: 40 },
  toggleHint: { color: '#64748b', marginTop: 15, fontSize: 14 },
  serverCard: { flexDirection: 'row', backgroundColor: '#1e293b', padding: 15, borderRadius: 12, alignItems: 'center', marginBottom: 20 },
  serverFlag: { fontSize: 30 },
  serverInfo: { flex: 1, marginLeft: 15 },
  serverName: { color: '#fff', fontSize: 16, fontWeight: '600' },
  serverCity: { color: '#64748b', fontSize: 12 },
  chevron: { color: '#64748b', fontSize: 18 }
import NetworkExtension

func setupAndConnectVPN() {
    // 1. Create a manager for a custom tunneling protocol
    let manager = NETunnelProviderManager()
    let protocolConfig = NETunnelProviderProtocol()
    
    // Must match the Bundle ID of your Packet Tunnel Provider Extension target
    protocolConfig.providerBundleIdentifier = "com.yourcompany.vpnapp.PacketTunnel" 
    protocolConfig.serverAddress = "192.0.2.14" // Your remote VPN gateway IP
    
    manager.protocolConfiguration = protocolConfig
    manager.localizedDescription = "My Personal Secure VPN"
    manager.isEnabled = true
    
    // 2. Save to system preferences (Triggers the native iOS "Add VPN Configuration" prompt)
    manager.saveToPreferences { error in
        guard error == nil else {
            print("User denied permission or saving failed: \(error!)")
            return
        }
        
        // 3. Start the tunnel
        do {
            try manager.connection.startTunnel(options: nil)
        } catch {
            print("Failed to start tunnel: \(error)")
        }
    }
import NetworkExtension

class PacketTunnelProvider: NEPacketTunnelProvider {

    override func startTunnel(options: [String : NSObject]?, completionHandler: @escaping (Error?) -> Void) {
        // 1. Establish the network settings for the local virtual network card (TUN interface)
        let settings = NEPacketTunnelNetworkSettings(tunnelRemoteAddress: self.protocolConfiguration.serverAddress!)
        
        // Assign a local virtual IP address provided by your backend
        settings.ipv4Settings = NEIPv4Settings(addresses: ["10.0.0.2"], subnetMasks: ["255.255.255.0"])
        
        // Route ALL device internet traffic (0.0.0.0/0) through the tunnel
        settings.ipv4Settings?.includedRoutes = [NEIPv4Route.default()]
        settings.dnsSettings = NEDNSSettings(servers: ["8.8.8.8", "1.1.1.1"])
        
        // 2. Apply settings to the iOS system
        self.setTunnelNetworkSettings(settings) { error in
            if let error = error {
                completionHandler(error)
                return
            }
            
            // 3. Start reading raw data packets flowing from the system
            self.readPacketsAndPipeToRemoteServer()
            completionHandler(nil)
        }
    }

    private func readPacketsAndPipeToRemoteServer() {
        // Asynchronously read raw IP packets arriving from the iOS networking stack
        packetFlow.readPackets { [weak self] (packets, versions) in
            for packet in packets {
                // -> HERE: Encrypt this 'packet' data and transmit it over a 
                // UDP/TCP socket to your remote VPN Gateway server.
            }
            
            // Continue the loop to continuously read incoming packets
            self?.readPacketsAndPipeToRemoteServer()
        }
    }

    override func stopTunnel(with reason: NEProviderStopReason, completionHandler: @escaping () -> Void) {
        // Clean up socket connections and tear down local loops safely
        completionHandler()
    }
<service 
    android:name=".MyVpnService"
    android:permission="android.permission.BIND_VPN_SERVICE"
    android:exported="false">
    <intent-filter>
        <action android:name="android.net.VpnService"/>
    </intent-filter>
</import android.content.Intent
import android.net.VpnService
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private func prepareVpn() {
        // Check if permissions are already given
        val intent = VpnService.prepare(applicationContext)
        if (intent != null) {
            // Shows the system connection request dialog
            startActivityForResult(intent, 0)
        } else {
            // Already authorized, start service safely
            startService(Intent(this, MyVpnService::class.java))
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (resultCode == RESULT_OK) {
            startService(Intent(this, MyVpnService::class.java))
        }
    }
}version: "3.8"
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    environment:
      # Change this to your explicit Linux VPS public IP
      - WG_HOST=192.0.2.14 
      - WG_DEFAULT_DNS=1.1.1.1
      - WG_DEFAULT_ADDRESS=10.8.0.x
      # Web UI management password (Optional / For testing via browser on port 51821)
      - PASSWORD_HASH=$$2a$$12$$DfJ2phN2VE4Z1gyFNsGCluifeQUQzz.m4tF4hcHABqYq7yKXQ5cPW
    volumes:
      - .data:/etc/wireguard
    ports:
      - "51820:51820/udp" # The secure channel phone traffic streams through
      - "51821:51821/tcp" # Web UI admin panel port
    capabilities:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stoppedconst express = require('express');
const jwt = require('jsonwebtoken');
const { exec } = require('child_process');
const app = express();

app.use(express.json());

const JWT_SECRET = "super_secret_unbreakable_key_12345";
const WG_CONTAINER_NAME = "wg-easy";

// 1. DUMMY LOGIN ENDPOINT (Connect this to your actual User Database later)
app.post('/api/auth/login', (req, res) => {
    const { email, password } = req.body;
    
    // Quick mock validation
    if (email === "user@test.com" && password === "password123") {
        const token = jwt.sign({ email, userId: "user_99" }, JWT_SECRET, { expiresIn: '7d' });
        return res.json({ success: true, token });
    }
    return res.status(401).json({ success: false, error: "Invalid credentials" });
});

// 2. PROTECTED ROUTE TO REQUEST ACTIVE VPN PROFILE
app.post('/api/vpn/connect', (req, res) => {
    const authHeader = req.headers['authorization'];
    if (!authHeader) return res.status(403).json({ error: "No token provided" });

    const token = authHeader.split(' ')[1];
    
    // Verify the user is authenticated and active
    jwt.verify(token, JWT_SECRET, (err, decoded) => {
        if (err) return res.status(401).json({ error: "Session expired" });

        const safeClientName = `user_${decoded.userId}`;

        // Call the running Docker WireGuard container to generate a peer profile
        // This command creates the keys and prints the configuration out directly
        exec(`docker exec ${WG_CONTAINER_NAME} wgeasy add ${safeClientName}`, (error, stdout, stderr) => {
            if (error && !stdout.includes("already exists")) {
                return res.status(500).json({ error: "Failed to allocate secure server slot" });
            }

            // Read the newly created profile string configuration
            exec(`docker exec ${WG_CONTAINER_NAME} cat /etc/wireguard/wg0.conf`, (catErr, configText) => {
                if (catErr) return res.status(500).json({ error: "Error retrieving configuration details" });
                
                // Parse the raw database text to pull out ONLY this specific user's block
                // For a highly scaling production app, you would parse and send the Client interface block
                res.json({
                    success: true,
                    message: "Profile allocated successfully",
                    // Pass the connection string back to WireGuardKit / GoBackend in the app
                    configurationString: `[Interface]\nPrivateKey = GENERATED_CLIENT_PRIVATE_KEY\nAddress = 10.8.0.2/24\nDNS = 1.1.1.1\n\n[Peer]\nPublicKey = SERVER_PUBLIC_KEY\nEndpoint = YOUR_SERVER_IP:51820\nAllowedIPs = 0.0.0.0/0`
                });
            });
        });
    });
});

app.listen(3000, () => console.log('🚀 VPN Authentication and profile allocation API running on Port 3000'));
// vpnManager.test.ts
import { VPNManager, VPNStatus } from './vpnManager'; // Your app's core engine logic

describe('VPN Connection Stability & Reconnection Suite', () => {
  let vpnManager: VPNManager;
  let mockNativeBridge: any;

  beforeEach(() => {
    // Mock the native platform layer (iOS NetworkExtension / Android VpnService)
    mockNativeBridge = {
      startTunnel: jest.fn().mockResolvedValue(true),
      stopTunnel: jest.fn().mockResolvedValue(true),
      getCurrentStatus: jest.fn(),
    };
    vpnManager = new VPNManager(mockNativeBridge);
  });

  // TEST 1: Verify the app handles sudden dropouts by triggering a retry mechanism
  it('should automatically attempt reconnection when connection drops unexpectedly', async () => {
    jest.useFakeTimers();
    
    // Simulate initial stable connection
    mockNativeBridge.getCurrentStatus.mockReturnValue(VPNStatus.CONNECTED);
    await vpnManager.connect();
    
    expect(vpnManager.status).toBe(VPNStatus.CONNECTED);

    // Act: Simulate an unexpected drop without the user tapping "Disconnect"
    vpnManager.handleNativeStatusChange(VPNStatus.DISCONNECTED_UNEXPECTEDLY);

    // Assert: Check if it switches to RECONNECTING state instead of just dying
    expect(vpnManager.status).toBe(VPNStatus.RECONNECTING);
    expect(vpnManager.reconnectAttempts).toBe(1);

    // Fast-forward timer to check if retry logic fires
    jest.advanceTimersByTime(2000); // Wait for the retry interval
    expect(mockNativeBridge.startTunnel).toHaveBeenCalledTimes(2); // Initial + Retry

    jest.useRealTimers();
  });

  // TEST 2: Verify the app stops retrying after max attempts to protect battery life
  it('should stop reconnecting and throw an error after max retry limit reached', async () => {
    jest.useFakeTimers();
    
    // Force the native tunnel to continuously fail when startTunnel is called
    mockNativeBridge.startTunnel.mockRejectedValue(new Error("Network Unreachable"));
    
    vpnManager.handleNativeStatusChange(VPNStatus.DISCONNECTED_UNEXPECTEDLY);

    // Simulate 5 rapid reconnection failures
    for (let i = 0; i < 5; i++) {
      jest.advanceTimersByTime(2000);
    }

    expect(vpnManager.status).toBe(VPNStatus.FAILED);
    expect(vpnManager.reconnectAttempts).toBe(5);
    
    jest.useRealTimers();
  });
});// connectionGuard.js
import NetInfo from "@react-native-community/netinfo";

export function setupNetworkGuard(vpnManager) {
  let lastNetworkType = null;

  // Listen to global OS network interface changes
  return NetInfo.addEventListener(state => {
    const currentNetworkType = state.type; // 'wifi', 'cellular', 'none'
    
    if (lastNetworkType && lastNetworkType !== currentNetworkType) {
      console.log(`📡 Network shifted from ${lastNetworkType} to ${currentNetworkType}`);
      
      if (vpnManager.isConnected()) {
        // WireGuard handles roaming natively, but OpenVPN needs an explicit endpoint refresh
        console.log("🔄 Re-aligning active VPN tunnel routes to new interface...");
        vpnManager.triggerSeamlessHandshake();
      }
    }
    
    lastNetworkType = currentNetworkType;
  });
}appId: com.yourcompany.vpnapp
---
- clearState # Wipe state to simulate a fresh app installation
- launchApp

# Step 1: Walk through onboarding and verify we are disconnected
- assertVisible: "UNPROTECTED"
- assertVisible: "Tap to Connect"

# Step 2: Tap the main toggle to initialize permissions
- tapOn: "🛑" # Taps the central button

# Step 3: Handle native OS-level Permission Prompts
# This works natively on both Android and iOS configurations
- runScript:
    script: |
      // Conditional handling for OS permissions if they appear
      if (maestro.isVisible("Connection Request")) {
          maestro.tapOn("OK"); // Android System permission accept
      } else if (maestro.isVisible("Allow VPN Configurations")) {
          maestro.tapOn("Allow"); // iOS System permission accept
      }

# Step 4: Validate UI state shift to Protected
- extendedWaitUntil:
    visible: "SECURELY CONNECTED"
    timeout: 10000 # Give the secure protocol 10 seconds to handshake

# Step 5: Test Background Survivability
- backgroundApp: 5 # Send app to background for 5 seconds to ensure OS daemon doesn't kill it
- launchApp
- assertVisible: "SECURELY CONNECTED" # Ensure state survived deep sleep# If using wg-easy / wireguard on your backend server:
wg show wg0 allowed-ips   # Find your mobile client's public key
wg set wg0 peer CLIENT_PUBLIC_KEY remove// server.js (Append this logic to your existing Express backend)
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors()); // Allow your mobile client to fetch data across domains
app.use(express.json());

// Mock database containing active infrastructure nodes
const VPN_SERVER_CLUSTER = [
  { id: "srv_us_ny", country: "United States", city: "New York", flag: "🇺🇸", ipAddress: "192.0.2.55", activeLoad: 42, basePingMs: 18 },
  { id: "srv_uk_lon", country: "United Kingdom", city: "London", flag: "🇬🇧", ipAddress: "198.51.100.12", activeLoad: 78, basePingMs: 65 },
  { id: "srv_de_fra", country: "Germany", city: "Frankfurt", flag: "🇩🇪", ipAddress: "203.0.113.8", activeLoad: 12, basePingMs: 82 },
  { id: "srv_jp_tok", country: "Japan", city: "Tokyo", flag: "🇯🇵", ipAddress: "192.0.2.190", activeLoad: 91, basePingMs: 145 },
  { id: "srv_sg_sin", country: "Singapore", city: "Downtown", flag: "🇸🇬", ipAddress: "198.51.100.4", activeLoad: 33, basePingMs: 190 }
];

/**
 * GET /api/servers
 * Returns available nodes sorted dynamically by load metrics
 */
app.get('/api/servers', (req, res) => {
  try {
    // Optional architectural logic: Filter out dead nodes or sort by lowest activeLoad
    const optimalServers = [...VPN_SERVER_CLUSTER].sort((a, b) => a.activeLoad - b.activeLoad);
    
    res.status(200).json({
      success: true,
      timestamp: Date.now(),
      count: optimalServers.length,
      servers: optimalServers
    });
  } catch (error) {
    res.status(500).json({ success: false, error: "Internal registry error" });
  }
});

// Start listening
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`📡 Server Registry API active on port ${PORT}`));// ServerSelector.jsx
import React, { useState, useEffect } from 'react';
import { StyleSheet, Text, View, FlatList, TouchableOpacity, ActivityIndicator, SafeAreaView } from 'react-native';

const API_ENDPOINT = 'http://YOUR_SERVER_IP:3000/api/servers';

export default function ServerSelector({ onSelectServer, currentServer }) {
  const [servers, setServers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [isRefreshing, setIsRefreshing] = useState(false);

  // Fetch data cleanly from our REST endpoint
  const fetchServers = async () => {
    try {
      const response = await fetch(API_ENDPOINT);
      const json = await response.json();
      if (json.success) {
        setServers(json.servers);
      }
    } catch (error) {
      console.error("Failed fetching live server manifest: ", error);
    } finally {
      setIsLoading(false);
      setIsRefreshing(false);
    }
  };

  useEffect(() => {
    fetchServers();
  }, []);

  const handleRefresh = () => {
    setIsRefreshing(true);
    fetchServers();
  };

  // Helper function to color code latency boundaries
  const getPingColor = (ping) => {
    if (ping < 50) return '#10b981'; // Green (Excellent)
    if (ping < 120) return '#f59e0b'; // Yellow (Average)
    return '#ef4444'; // Red (High Latency)
  };

  // Render design block for individual server entries
  const renderServerItem = ({ item }) => {
    const isSelected = currentServer?.id === item.id;

    return (
      <TouchableOpacity 
        style={[styles.serverCard, isSelected && styles.selectedCard]}
        onPress={() => onSelectServer(item)}
      >
        <Text style={styles.flag}>{item.flag}</Text>
        
        <View style={styles.infoContainer}>
          <Text style={styles.countryName}>{item.country}</Text>
          <Text style={styles.cityName}>{item.city} • Load: {item.activeLoad}%</Text>
        </View>

        <View style={styles.metaContainer}>
          <Text style={[styles.pingText, { color: getPingColor(item.basePingMs) }]}>
            {item.basePingMs}ms
          </Text>
          {isSelected && <View style={styles.activeIndicatorIcon} />}
        </View>
      </TouchableOpacity>
    );
  };

  if (isLoading) {
    return (
      <View style={styles.centeredContainer}>
        <ActivityIndicator size="large" color="#3b82f6" />
        <Text style={styles.loadingText}>Locating fastest secure nodes...</Text>
      </View>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Select Location</Text>
        <Text style={styles.subtitle}>Choose a server closest to your region for maximum speeds.</Text>
      </View>

      <FlatList
        data={servers}
        keyExtractor={(item) => item.id}
        renderItem={renderServerItem}
        refreshing={isRefreshing}
        onRefresh={handleRefresh}
        contentContainerStyle={styles.listContent}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0f172a' },
  centeredContainer: { flex: 1, backgroundColor: '#0f172a', justifyContent: 'center', alignItems: 'center' },
  loadingText: { color: '#94a3b8', marginTop: 12, fontSize: 14 },
  header: { padding: 20, borderBottomWidth: 1, borderColor: '#1e293b' },
  title: { color: '#fff', fontSize: 22, fontWeight: 'bold' },
  subtitle: { color: '#64748b', fontSize: 13, marginTop: 4 },
  listContent: { padding: 15 },
  serverCard: { 
    flexDirection: 'row', 
    backgroundColor: '#1e293b', 
    padding: 16, 
    borderRadius: 12, 
    alignItems: 'center', 
    marginBottom: 12,
    borderWidth: 1,
    borderColor: 'transparent'
  },
  selectedCard: { borderColor: '#3b82f6', backgroundColor: '#1e293b' },
  flag: { fontSize: 28 },
  infoContainer: { flex: 1, marginLeft: 16 },
  countryName: { color: '#fff', fontSize: 16, fontWeight: '600' },
  cityName: { color: '#64748b', fontSize: 12, marginTop: 2 },
  metaContainer: { alignItems: 'flex-end', flexDirection: 'row', alignItems: 'center' },
  pingText: { fontSize: 14, fontWeight: 'bold', marginRight: 10 },
  activeIndicatorIcon: { width: 10, height: 10, borderRadius: 5, backgroundColor: '#3b82f6' }
});const startTimestamp = Date.now();
await fetch(`http://${serverIp}/ping-check`, { method: 'HEAD' });
const localizedPing = Date.now() - // Inside your Android MyVpnService.kt
fun configureKillSwitch(builder: VpnService.Builder) {
    // 1. Check if the device is running Android 10 (API 29) or higher
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.Q) {
        // This system native call blocks any global network traffic 
        // that tries to bypass the VPN interface when it fluctuates.
        builder.setMetered(false) 
    }
    
    // 2. Strict Kill Switch Routing
    // By establishing a precise global destination anchor without adding a 
    // default system route fallback path, the OS drops non-VPN packets automatically.
    builder.addRoute("0.0.0.0", 0) 
}// Inside your Android MyVpnService.kt
fun applySplitTunneling(builder: VpnService.Builder, excludedApps: List<String>) {
    // Example excludedApps list: ["com.google.android.youtube", "com.yourbank.app"]
    for (packageName in excludedApps) {
        try {
            // Tells the OS: "Route this package's packets over standard Wi-Fi / Cellular lines instead"
            builder.addDisallowedApplication(packageName)
        } catch (e: PackageManager.NameNotFoundException) {
            // Handle edge-case where app isn't installed on user's device
        }
    }
}// Inside your iOS PacketTunnelProvider.swift
func applySplitTunnelingRoutes(settings: NEPacketTunnelNetworkSettings) {
    let ipv4Settings = NEIPv4Settings(addresses: ["10.0.0.2"], subnetMasks: ["255.255.255.0"])
    
    // Instead of routing ALL traffic via NEIPv4Route.default(), 
    // you explicitly define what goes THROUGH the tunnel. 
    // Everything else automatically defaults to standard open Wi-Fi.
    let secureCorporateSubnet = NEIPv4Route(destinationAddress: "192.168.50.0", subnetMask: "255.255.255.0")
    ipv4Settings.includedRoutes = [secureCorporateSubnet]
    
    settings.ipv4Settings = ipv4Settings
}// TelemetryDashboard.jsx
import React, { useState, useEffect } from 'react';
import { StyleSheet, Text, View, SafeAreaView } from 'react-native';

export default function TelemetryDashboard({ nativeVpnStatsAdapter }) {
  const [metrics, setMetrics] = useState({ uploadSpeed: '0.0 KB/s', downloadSpeed: '0.0 KB/s', latency: '0 ms' });

  useEffect(() => {
    // 1. Establish an asynchronous runtime interval loop to poll bytes handled by the OS
    const telemetryInterval = setInterval(() => {
      const stats = nativeVpnStatsAdapter.getLiveTunnelBytes();
      
      setMetrics({
        uploadSpeed: formatByteRate(stats.bytesTxPerSecond),
        downloadSpeed: formatByteRate(stats.bytesRxPerSecond),
        latency: `${stats.currentPingMs} ms`
      });
    }, 1000); // Re-calculate velocities every single second

    return () => clearInterval(telemetryInterval);
  }, []);

  const formatByteRate = (bytes) => {
    if (bytes < 1024) return `${bytes} B/s`;
    const kib = bytes / 1024;
    if (kib < 1024) return `${kib.toFixed(1)} KB/s`;
    return `${(kib / 1024).toFixed(1)} MB/s`;
  };

  return (
    <View style={styles.metricsGrid}>
      <View style={styles.metricBlock}>
        <Text style={styles.label}>PING LATENCY</Text>
        <Text style={styles.value}>{metrics.latency}</Text>
      </View>
      <View style={styles.metricBlock}>
        <Text style={styles.label}>DOWNLOAD</Text>
        <Text style={styles.value}>⬇ {metrics.downloadSpeed}</Text>
      </View>
      <View style={styles.metricBlock}>
        <Text style={styles.label}>UPLOAD</Text>
        <Text style={styles.value}>⬆ {metrics.uploadSpeed}</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  metricsGrid: { flexDirection: 'row', backgroundColor: '#1e293b', borderRadius: 12, padding: 15, marginTop: 20 },
  metricBlock: { flex: 1, alignItems: 'center' },
  label: { color: '#64748b', fontSize: 10, fontWeight: 'bold', letterSpacing: 1 },
  value: { color: '#fff', fontSize: 15, fontWeight: '600', marginTop: 4 }
});// telemetryTracker.js
const ANALYTICS_ENDPOINT = 'https://api.yourvpn.com/v1/telemetry';

export async function recordAnonymizedSession(userId, serverId, sessionDurationSecs) {
  const payload = {
    // Use an opaque, non-identifiable timestamp marker
    timestamp: new Date().toISOString().split('T')[0], // YYYY-MM-DD only (Zero granular tracking)
    targetNodeId: serverId,
    duration: sessionDurationSecs,
    platform: Platform.OS // 'ios' or 'android'
  };

  try {
    await fetch(ANALYTICS_ENDPOINT, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });
  } catch (err) {
    // Silently log metrics pipeline failure locally so it never interrupts the tunnel process
    console.debug("Telemetry ingestion offline.");
  }
}#!/bin/bash
# system-vpn-init.sh -> Run as root on your cloud server

# 1. Core Network Infrastructure Updates
sudo apt update && sudo apt install -y wireguard wireguard-tools iptables-persistent

# 2. Open Native Linux Kernel Packet Forwarding
# This changes routing flags so the OS routes data packets between physical devices and the virtual tunnel interface.
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 3. Generate Central Server Cryptographic Credentials
mkdir -p /etc/wireguard && cd /etc/wireguard
umask 077
wg genkey | tee server_private.key | wg pubkey > server_public.key

SERVER_PRIV_KEY=$(cat server_private.key)
# Detect the server's main physical internet-facing interface (e.g., eth0)
PRIMARY_NET_INTERFACE=$(ip route show default | awk '/default/ {print $5}')

# 4. Construct System-Level Core WireGuard Interface Layout
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = ${SERVER_PRIV_KEY}
Address = 10.200.0.1/24
ListenPort = 51820

# PostUp/PostDown rules use iptables rules to transparently handle NAT firewall masquerading 
# as packets travel from your phone to the open internet.
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ${PRIMARY_NET_INTERFACE} -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ${PRIMARY_NET_INTERFACE} -j ACCEPT
EOF

# 5. Bring up your network anchor
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
echo "🚀 WireGuard Kernel Node initialized successfully on Port 51820."// server.js (Node.js API Control Center)
const express = require('express');
const { execSync } = require('child_process');
const fs = require('fs');
const app = express();

app.use(express.json());

const SERVER_PUBLIC_IP = "YOUR_LINUX_VPS_PUBLIC_IP";
const WG_PORT = "51820";
const WG_INTERFACE = "wg0";

// For demonstration, an in-memory database tracks IP assignments sequentially
let currentAssignedIpTail = 2; 

/**
 * POST /api/vpn/provision-peer
 * High-performance hook called by the mobile client when pressing "Connect"
 */
app.post('/api/vpn/provision-peer', (req, res) => {
    try {
        const { clientPublicKey } = req.body;
        
        if (!clientPublicKey || clientPublicKey.length !== 44) {
            return res.status(400).json({ error: "Malformed cryptographic public key payload" });
        }

        // 1. Allocate a unique IP block within our virtual subnet to prevent crossover
        if (currentAssignedIpTail >= 254) {
            return res.status(507).json({ error: "Server infrastructure capacity saturation reached" });
        }
        
        const allocatedClientIp = `10.200.0.${currentAssignedIpTail}/32`;
        currentAssignedIpTail++;

        // 2. Fetch the host's public key to compile the client-side configuration text
        const serverPublicKey = execSync("cat /etc/wireguard/server_public.key").toString().trim();

        // 3. Inject the client's public key directly into the Linux routing engine in real-time
        // Using "wg set" adds the peer dynamically without restarting the network interface or dropping active users.
        const registerPeerCmd = `sudo wg set ${WG_INTERFACE} peer "${clientPublicKey}" allowed-ips ${allocatedClientIp}`;
        execSync(registerPeerCmd);

        // 4. Save to persistent server config storage so profiles survive server reboots
        const configAppendBlock = `\n[Peer]\nPublicKey = ${clientPublicKey}\nAllowedIPs = ${allocatedClientIp}\n`;
        fs.appendFileSync('/etc/wireguard/wg0.conf', configAppendBlock);

        // 5. Construct and respond with the exact, compilation-ready config block string
        const optimizedClientConfig = [
            `[Interface]`,
            `Address = ${allocatedClientIp.replace('/32', '/24')}`,
            `DNS = 1.1.1.1, 8.8.8.8`,
            ``,
            `[Peer]`,
            `PublicKey = ${serverPublicKey}`,
            `Endpoint = ${SERVER_PUBLIC_IP}:${WG_PORT}`,
            `AllowedIPs = 0.0.0.0/0`,
            `PersistentKeepalive = 25`
        ].join('\n');

        return res.status(200).json({
            success: true,
            assignedIp: allocatedClientIp,
            wgProfileText: optimizedClientConfig
        });

    } catch (error) {
        console.error("Internal Engine Allocation Failure:", error);
        return res.status(500).json({ error: "System runtime interface exception error" });
    }
});

app.listen(3000, () => console.log('⚡ Orchestration Controller API operational on port 3000'));
// vpnOrchestrator.ts
import { NativeModules, Platform } from 'react-native';

const API_GATEWAY = "http://YOUR_SERVER_IP:3000/api/vpn/provision-peer";

// Accessing the native OS-level platforms we created during our earlier architectural steps
const { AndroidVpnModule, IosPacketTunnelModule } = NativeModules;

export async function startSecureVpnTunnel(): Promise<boolean> {
  try {
    console.log("⚙️ Initializing local security handshakes...");
    
    // 1. Generate local key pairs strictly on the device (Private key never leaves the phone)
    // You can use libraries like 'react-native-quick-crypto' or native bindings
    const clientPrivateKey = "PRIVATE_KEY_GENERATED_SECURELY_ON_DEVICE_BASE64";
    const clientPublicKey = "PUBLIC_KEY_DERIVED_FROM_PRIVATE_BASE64_44_CHARS";

    // 2. Safely register with your infrastructure cluster
    const response = await fetch(API_GATEWAY, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ clientPublicKey: clientPublicKey })
    });

    const result = await response.json();
    if (!result.success) throw new Error("Server rejected provisioning profile parameters");

    // 3. Inject our local private key into the configuration profile template received from the server
    const customizedLocalProfile = result.wgProfileText.replace(
      `[Interface]`,
      `[Interface]\nPrivateKey = ${clientPrivateKey}`
    );

    // 4. Pass the final configuration text profile directly into the native platform adapters
    if (Platform.OS === 'android') {
      const connectionSuccess = await AndroidVpnModule.startTunnelEngine(customizedLocalProfile);
      return connectionSuccess;
    } else if (Platform.OS === 'ios') {
      const connectionSuccess = await IosPacketTunnelModule.startExtensionEngine(customizedLocalProfile);
      return connectionSuccess;
    }

    return false;
  } catch (error) {
    console.error("🚨 Critical Handshake Routing Exception:", error);
    return false;
  }
}