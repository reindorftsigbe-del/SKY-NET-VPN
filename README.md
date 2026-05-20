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
