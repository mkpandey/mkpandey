import software.amazon.awssdk.auth.credentials.ProfileCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.*;

import java.time.Instant;
import java.time.LocalDate;
import java.time.ZoneId;
import java.util.ArrayList;
import java.util.List;

public class S3FileDeletionBatch {

    private static final Region AWS_REGION = Region.US_EAST_1;
    private static final String BUCKET_NAME = "your-bucket-name";

    public static void main(String[] args) {
        S3Client s3Client = S3Client.builder()
                .region(AWS_REGION)
                .credentialsProvider(ProfileCredentialsProvider.create())
                .build();

        LocalDate oneMonthAgo = LocalDate.now().minusMonths(1);

        List<String> keysToDelete = new ArrayList<>();

        ListObjectsV2Request listRequest = ListObjectsV2Request.builder()
                .bucket(BUCKET_NAME)
                .build();

        ListObjectsV2Response listResponse;
        do {
            listResponse = s3Client.listObjectsV2(listRequest);

            for (S3Object object : listResponse.contents()) {
                LocalDate lastModified = Instant.ofEpochMilli(object.lastModified().toEpochMilli())
                        .atZone(ZoneId.systemDefault())
                        .toLocalDate();

                if (lastModified.isBefore(oneMonthAgo)) {
                    keysToDelete.add(object.key());
                }
            }

            listRequest = listRequest.toBuilder()
                    .continuationToken(listResponse.nextContinuationToken())
                    .build();

        } while (listResponse.isTruncated());

        deleteObjects(s3Client, BUCKET_NAME, keysToDelete);

        s3Client.close();
    }

    private static void deleteObjects(S3Client s3Client, String bucketName, List<String> keysToDelete) {
        if (!keysToDelete.isEmpty()) {
            DeleteObjectsRequest deleteRequest = DeleteObjectsRequest.builder()
                    .bucket(bucketName)
                    .delete(Delete.builder().objects(keysToDelete.stream().map(DeleteObject.builder()::key).build()).build())
                    .build();

            DeleteObjectsResponse deleteResponse = s3Client.deleteObjects(deleteRequest);

            // Check if any objects failed to delete
            if (!deleteResponse.deletedObjects().isEmpty()) {
                System.out.println("Successfully deleted objects:");
                deleteResponse.deletedObjects().forEach(deletedObject -> System.out.println(deletedObject.key()));
            }

            // Check if any objects failed to delete
            if (!deleteResponse.errors().isEmpty()) {
                System.out.println("Failed to delete objects:");
                deleteResponse.errors().forEach(error -> System.out.println(error.message()));
            }
        } else {
            System.out.println("No objects found older than one month.");
        }
    }
}


 DeleteObjectsRequest deleteRequest = DeleteObjectsRequest.builder()
                .bucket(bucketName)
                .delete(Delete.builder().objects(keysToDelete).build())
                .build();
