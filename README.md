import software.amazon.awssdk.auth.credentials.ProfileCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.DeleteObjectRequest;
import software.amazon.awssdk.services.s3.model.ListObjectsV2Request;
import software.amazon.awssdk.services.s3.model.ListObjectsV2Response;
import software.amazon.awssdk.services.s3.model.S3Object;

import java.time.Instant;
import java.time.LocalDate;
import java.time.ZoneId;
import java.util.Date;

public class S3FileDeletion {

    private static final Region AWS_REGION = Region.US_EAST_1;
    private static final String BUCKET_NAME = "your-bucket-name";
    private static final String PREFIX = "logs/"; // Example prefix

    public static void main(String[] args) {
        S3Client s3 = S3Client.builder()
                .region(AWS_REGION)
                .credentialsProvider(ProfileCredentialsProvider.create())
                .build();

        LocalDate oneMonthAgo = LocalDate.now().minusMonths(1);

        ListObjectsV2Request listReq = ListObjectsV2Request.builder()
                .bucket(BUCKET_NAME)
                .prefix(PREFIX)
                .build();

        ListObjectsV2Response listRes;
        do {
            listRes = s3.listObjectsV2(listReq);

            for (S3Object s3Object : listRes.contents()) {
                LocalDate lastModified = Instant.ofEpochMilli(s3Object.lastModified().toEpochMilli())
                        .atZone(ZoneId.systemDefault())
                        .toLocalDate();

                if (lastModified.isBefore(oneMonthAgo)) {
                    deleteS3Object(s3, BUCKET_NAME, s3Object.key());
                }
            }

            listReq = listReq.toBuilder()
                    .continuationToken(listRes.nextContinuationToken())
                    .build();

        } while (listRes.isTruncated());

        s3.close();
    }

    private static void deleteS3Object(S3Client s3, String bucketName, String key) {
        DeleteObjectRequest deleteRequest = DeleteObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();

        s3.deleteObject(deleteRequest);
        System.out.println("Deleted object: " + key);
    }
}
