cat << 'EOF' > /tmp/run-lab.sh
#!/bin/bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
export ENDPOINT="http://127.0.0.1:4566"

echo "--- Phase 1: S3 Bucket Operations ---"
aws --endpoint-url=$ENDPOINT s3 mb s3://cyber-lab-data 2>/dev/null || true
echo "CONFIDENTIAL CYBER LAB DATA" > /tmp/cloud-secret.txt
aws --endpoint-url=$ENDPOINT s3 cp /tmp/cloud-secret.txt s3://cyber-lab-data/

echo "--- Phase 2: IAM Identity & Access Management ---"
aws --endpoint-url=$ENDPOINT iam create-user --user-name cloud-lab-user 2>/dev/null || true
cat > /tmp/s3-lab-policy.json << 'INNER_EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "LabS3FullAccess",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
INNER_EOF
aws --endpoint-url=$ENDPOINT iam create-policy --policy-name LabS3FullAccess --policy-document file:///tmp/s3-lab-policy.json 2>/dev/null || true
aws --endpoint-url=$ENDPOINT iam attach-user-policy --user-name cloud-lab-user --policy-arn arn:aws:iam::000000000000:policy/LabS3FullAccess 2>/dev/null || true
aws --endpoint-url=$ENDPOINT iam create-access-key --user-name cloud-lab-user 2>/dev/null || true

echo "--- Phase 3: CloudTrail Audit Logging ---"
aws --endpoint-url=$ENDPOINT s3 mb s3://cloudtrail-lab-logs 2>/dev/null || true
aws --endpoint-url=$ENDPOINT cloudtrail create-trail --name cloud-security-lab-trail --s3-bucket-name cloudtrail-lab-logs 2>/dev/null || true
aws --endpoint-url=$ENDPOINT cloudtrail start-logging --name cloud-security-lab-trail 2>/dev/null || true

echo "--- Phase 4: Detection & Verification ---"
echo "CLOUDTRAIL DETECTION TEST $(date)" > /tmp/cloudtrail-event-test.txt
aws --endpoint-url=$ENDPOINT s3 cp /tmp/cloudtrail-event-test.txt s3://cyber-lab-data/
aws --endpoint-url=$ENDPOINT s3 rm s3://cyber-lab-data/cloudtrail-event-test.txt
aws --endpoint-url=$ENDPOINT s3api create-bucket --bucket cloudtrail-event-test-001 2>/dev/null || true

echo "--- Phase 5: Public S3 Bucket Policy ---"
aws --endpoint-url=$ENDPOINT s3 mb s3://cyber-lab-public 2>/dev/null || true
echo "PUBLIC CLOUD SECURITY LAB TEST" > /tmp/public-test.txt
aws --endpoint-url=$ENDPOINT s3 cp /tmp/public-test.txt s3://cyber-lab-public/
cat > /tmp/public-s3-policy.json << 'INNER_EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadLab",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cyber-lab-public/*"
    }
  ]
}
INNER_EOF
aws --endpoint-url=$ENDPOINT s3api put-bucket-policy --bucket cyber-lab-public --policy file:///tmp/public-s3-policy.json 2>/dev/null || true

echo "--- Lab Execution Complete ---"
EOF
chmod +x /tmp/run-lab.sh
/tmp/run-lab.sh