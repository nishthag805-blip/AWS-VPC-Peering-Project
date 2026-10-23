# Example route table entries
VPC1_Private_RouteTable:
 - Destination: 10.1.0.0/16, Target: vpc-peering-connection-01234567

VPC2_Private_RouteTable:
 - Destination: 10.0.0.0/16, Target: vpc-peering-connection-01234567
resource "aws_vpc" "VPC1" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "VPC1" }
}
resource "aws_vpc" "VPC2" {
  cidr_block = "10.1.0.0/16"
  tags = { Name = "VPC2" }
}

# Create subnets, IGWs, NATs, etc... (omitted for brevity)

resource "aws_vpc_peering_connection" "Peer" {
  vpc_id        = aws_vpc.VPC1.id
  peer_vpc_id   = aws_vpc.VPC2.id
  auto_accept   = true
  tags = { Name = "VPC1-VPC2-Peering" }
}

resource "aws_route" "RouteToVPC2" {
  route_table_id         = aws_route_table.VPC1_Private.id
  destination_cidr_block = aws_vpc.VPC2.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.Peer.id
}
resource "aws_route" "RouteToVPC1" {
  route_table_id         = aws_route_table.VPC2_Private.id
  destination_cidr_block = aws_vpc.VPC1.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.Peer.id
}


#!/usr/bin/env bash

# AWS VPC Peering Connectivity Test
# Tests communication between two private EC2 instances.

set -u

PEER_IP="${1:-}"

if [[ -z "$PEER_IP" ]]; then
    echo "Usage: ./test-connectivity.sh <peer-private-ip>"
    echo "Example: ./test-connectivity.sh 10.20.1.10"
    exit 1
fi

echo "========================================"
echo "     AWS VPC Peering Connectivity Test"
echo "========================================"

echo "Current Host: $(hostname)"
echo "Peer IP: $PEER_IP"
echo ""

echo "Testing connectivity..."
echo ""

if ping -c 4 -W 2 "$PEER_IP"; then
    echo ""
    echo "SUCCESS: Peer is reachable."
else
    echo ""
    echo "FAILED: Peer is unreachable."
    echo "Check VPC peering, route tables, security groups,"
    echo "network ACLs, and host firewall settings."
    exit 1
fi

