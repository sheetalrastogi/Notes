Selenium + TestNG Custom Annotations Classification Matrix
#	Custom Annotation	Objective	Purpose	Sample Usage1	Retry Control	Control retry behavior at test level	Retry failed tests; Override default retry count	@RetryCount(3)
2	Browser Selection	Define browser per test	Execute on specific browser; Override suite browser	@Browser("chrome")
3	Requirement Traceability	Map tests to requirements	RTM generation; Compliance audits; Coverage tracking	@Requirement("REQ-1001")
4	Defect Mapping	Associate tests with known defects	Known bug tracking; Coverage gap analysis	@KnownIssue("BUG-4521")
5	Risk-Based Execution	Classify business risk	Prioritize testing; Risk-based regression	@RiskLevel("HIGH")
6	Test Owner Assignment	Identify accountable engineer	Ownership tracking; Failure notifications	@TestOwner("Sheetal")
7	Module / Feature Tagging	Group tests by business area	Module-wise execution; Report filtering	@Module("Claims")
8	Environment Restriction	Control execution environments	Restrict environments; Block PROD execution	@Environment({"QA","UAT"})
9	Smoke/Sanity/Regression Classification	Categorize test pack	Dynamic suite creation; Selective execution	@ExecutionType("SMOKE")
10	Feature Toggle Control	Link tests to feature flags	Execute only when feature enabled	@FeatureFlag("NewPaymentGateway")
11	Data Source Selection	Specify test data source	Dynamic data loading; Dataset management	@DataSource("PolicyData.xlsx")
12	Parallel Execution Eligibility	Mark parallel-safe tests	Prevent thread conflicts; Improve stability	@ThreadSafe(true)
13	Execution Priority	Define business execution order	Run critical tests first	@BusinessPriority(1)
14	SLA / Timeout Specification	Set custom timeout	Control long-running tests; SLA validation	@TimeoutSeconds(60)
15	Security Test Marker	Identify security validation tests	Security suite filtering; Audit reporting	@SecurityTest
16	Test Type Classification	Categorize automation type	Separate pipelines; Execution filtering	@TestType("API")
17	Performance Benchmark Metadata	Define expected performance target	SLA validation; Performance assertions	@ExpectedResponseTime(ms=2000)
18	Dependency Annotation	Define business dependencies	Workflow orchestration; Dependency tracking	@DependsOnBusinessProcess("CreatePolicy")
19	Auto-Healing Control	Control self-healing behavior	Enable AI locator healing; Per-test configuration	@SelfHealing(enabled=true)
20	Screenshot Capture Policy	Configure screenshot generation	Evidence collection; Failure analysis	@CaptureScreenshot(onFailure=true)
21	Video Recording Policy	Configure session recording	Execution playback; Failure debugging	@RecordExecutionVideo(true)
22	Accessibility Testing Marker	Tag accessibility validation tests	Accessibility suite execution; Compliance reporting	@AccessibilityTest
23	Flaky Test Tracking	Identify unstable tests	Flaky test analysis; Stability metrics	@FlakyTest
24	Production Safe Test	Mark PROD-executable tests	Prevent destructive actions; Safe production validation	@ProductionSafe
25	AI-Agent Driven Execution	Invoke framework agents dynamically	AI-driven execution; Intelligent automation	@AgentExecution("DomAnalysisAgent")
