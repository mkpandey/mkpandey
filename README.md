import org.junit.jupiter.api.Test;
import java.lang.reflect.Field;
import static org.junit.jupiter.api.Assertions.*;

public class MixedTypeModelTest {

    @Test
    void testMixedTypeModel() throws IllegalAccessException {
        MixedTypeModel model = new MixedTypeModel();

        // Retrieve all fields using reflection
        Field[] fields = model.getClass().getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true); // Ensure private fields are accessible

            // Get the initial value of the field
            Object initialValue = field.get(model);

            // Perform appropriate assertions based on field type
            if (initialValue instanceof String) {
                assertNull(initialValue);
                field.set(model, "Hello");
                assertEquals("Hello", field.get(model));
            } else if (initialValue instanceof Integer) {
                assertEquals(0, initialValue);
                field.set(model, 42);
                assertEquals(42, field.get(model));
            } else if (initialValue instanceof Boolean) {
                assertFalse((Boolean) initialValue);
                field.set(model, true);
                assertTrue((Boolean) field.get(model));
            } else {
                assertNull(initialValue);
                field.set(model, new Object());
                assertNotNull(field.get(model));
            }
        }
    }
}
