# Logstash Neo4j Integration

This repository contains a Logstash pipeline configuration for integrating with Neo4j Graph Database using the HTTP Query API.

## Repository Structure

```
logstash-neo4j-integration/
├── README.md
├── pipelines/
│   └── auth_neo4j_http_param.conf
```

## Prerequisites

- An Amazon EC2 instance running Amazon Linux 2
- Internet connectivity for downloading packages and connecting to Neo4j

## Installation Steps

### Install Logstash 8.17.3

Add Elastic repository:
```bash
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Create a file at `/etc/yum.repos.d/elastic.repo` with the following content:
```
[elastic-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Install Logstash:
```bash
sudo yum install logstash-8.17.3
```

Verify installation:
```bash
/usr/share/logstash/bin/logstash --version
```

## Create and Run Pipeline

Create the pipeline configuration file at `/etc/logstash/conf.d/auth_neo4j_http_param.conf`:

```
input {
  generator {
    count => 1
    lines => [ '{"name": "Yancarlo"}' ]
    codec => "json"
  }
}
output {
  stdout { codec => rubydebug }
  
  http {
    url => "https://<YOUR-INSTANCE-ID>.databases.neo4j.io/db/neo4j/query/v2"
    http_method => "post"
    headers => [
      "Authorization", "Basic YOUR_BASE64_STRING",
      "Content-Type", "application/json",
      "Accept", "application/json"
    ]
    format => "json"
    mapping => {
      "statement" => 'MERGE (n:Person {name: $name}) RETURN n'
      "parameters" => {
        "name" => "%{name}"
      }
    }
  }
}
```

Run the pipeline:
```bash
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/auth_neo4j_http_param.conf
```
Run the pipeline with debug mode:
```bash
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/auth_neo4j_http_param.conf --log.level=debug
```

## Understanding the Pipeline

### Pipeline Components

This pipeline consists of:

1. **Input Plugin - Generator**:
   - Creates a single event with a JSON document `{"name": "Yancarlo"}`
   - Uses the `json` codec to parse the event into a Logstash event

2. **Output Plugins**:
   - **stdout**: Outputs the event to the console using the `rubydebug` codec for debugging
   - **http**: Sends the event to Neo4j using its HTTP API

### HTTP Request Format

The HTTP request to Neo4j:
- Uses POST method to the transaction endpoint
- Contains authorization headers for authentication
- Formats the request as JSON with:
  - The Cypher statement 
  - Parameters that are substituted into the query

Pipeline execution flow:
1. Generates a single event containing a JSON document with a name field
2. Sends that data to Neo4j using the HTTP API
3. Creates or updates a Person node with the provided name
4. Also outputs the data to stdout for verification

## Neo4j Authentication

The pipeline uses Basic Authentication for Neo4j. To generate the Base64 encoded string:

1. Combine your Neo4j username and password in the format `username:password`
2. Encode this string in Base64

```bash
echo -n "username:password" | base64
```

Then use the encoded string in your pipeline configuration:
```
"Authorization", "Basic YOUR_BASE64_STRING"
```

## Resources

- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Neo4j Query API Documentation](https://neo4j.com/docs/query-api/current/)
- [Logstash HTTP Output Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-http.html)