Feature: MPID Validation

    Scenario: Submitter MPID equals Reporting party MPID
        Given a report with Submitter MPID = "12345"
        And Reporting party MPID = "12345"
        When the report is submitted
        Then the report should be accepted

    Scenario: Submitter MPID != Reporting party MPID, but valid agreement exists
        Given a report with Submitter MPID = "12345"
        And Reporting party MPID = "67890"
        And Covered person MPID = "67890"
        And a valid agreement exists between Submitter and Covered person
        When the report is submitted
        Then the report should be accepted

    Scenario: Unique MPIDs and valid agreement
        Given a report with Submitter MPID = "12345"
        And Reporting party MPID = "67890"
        And Covered person MPID = "98765"
        And a valid agreement exists between Submitter and Reporting person
        When the report is submitted
        Then the report should be accepted

    Scenario: Unique MPIDs and no agreement
        Given a report with Submitter MPID = "12345"
        And Reporting party MPID = "67890"
        And Covered person MPID = "98765"
        When the report is submitted
        Then the report should be rejected



        ====================================================================

        Feature: MPID Validation

    Scenario Outline: Validate report submission
        Given a report with Submitter MPID = "<submitter_mpid>"
        And Reporting party MPID = "<reporting_party_mpid>"
        And Covered person MPID = "<covered_person_mpid>"
        And a valid agreement exists between Submitter and Covered person: <agreement_exists_sc>
        And a valid agreement exists between Submitter and Reporting person: <agreement_exists_sr>
        When the report is submitted
        Then the report should <result>

    Examples:
        | submitter_mpid | reporting_party_mpid | covered_person_mpid | agreement_exists_sc | agreement_exists_sr | result      |
        | 12345         | 12345              | 12345              | true              | true              | accepted    |
        | 12345         | 67890              | 67890              | true              | false             | accepted    |
        | 12345         | 67890              | 98765              | true              | true              | accepted    |
        | 12345         | 67890              | 98765              | false             | false             | rejected    |

============================================================================================================================================

import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

public class ReportSteps {

    @Autowired
    private ReportService reportService;

    @Given("a report with Submitter MPID = \"{string}\"")
    public void a_report_with_Submitter_MPID(String submitterMpid) {
        context.report = new Report(submitterMpid, null, null);
    }

    @Given("Reporting party MPID = \"{string}\"")
    public void reporting_party_MPID(String reportingPartyMpid) {
        context.report.setReportingPartyMPID(reportingPartyMpid);
    }

    @Given("Covered person MPID = \"{string}\"")
    public void covered_person_MPID(String coveredPersonMpid) {
        context.report.setCoveredPersonMPID(coveredPersonMpid);
    }

    @Given("a valid agreement exists between Submitter and Covered person: {string}")
    public void a_valid_agreement_exists_between_Submitter_and_Covered_person(String agreementExists) {
        context.report.setAgreementSC(Boolean.parseBoolean(agreementExists));
    }

    @Given("a valid agreement exists between Submitter and Reporting person: {string}")
    public void a_valid_agreement_exists_between_Submitter_and_Reporting_person(String agreementExists) {
        context.report.setAgreementSR(Boolean.parseBoolean(agreementExists));
    }

    @When("the report is submitted")
    public void the_report_is_submitted() {
        context.response = reportService.submitReport(context.report);
    }

    @Then("the report should be accepted")
    public void the_report_should_be_accepted() {
        assert context.response.getStatusCode() == HttpStatus.OK;
    }

    @Then("the report should be rejected")
    public void the_report_should_be_rejected() {
        assert context.response.getStatusCode() == HttpStatus.BAD_REQUEST;
    }
}
